Jo Zhang  [1:19 AM]
Chatted with @Deep Mehta on the existing setup of the hub backend and why it is so slow & painful to maintain/update the workflows:

Right now, all the hub workflows are manually published from cloud interface, and lives in supabase, while in-app templates are published from the template repo.
Early on, the hub project had a higher priority and had the expectation for a large number of UGC creators/ new workflows to be published from the Cloud interface, so the initial pod decided to build a new database in the Cloud backend. @Christian Byrne did raise the concern of the triple publishing workload across local, cloud, hub.
When we cut the scope and lowered the priority of the hub, the site was already built from the cloud backend, and we had no resources to rebuild from the template repo again, so the half-built infra is more like a tech debt.
We had the plan to sync cloud template publishing from the hub backend but the work was never picked up by Si due to other priorities.
Right now, every time we change the spec for workflows, metadata, etc in backend, we need to wait for cloud deploy. Also all sorts of access gated in cloud.

---

In order to speed up the iteration process, the ideal case will actually be rebuilding the hub site from the repo, but maintaining the admin UI for the creators to submit, edit, and update workflows.
Or at least, we can try to finish point 4 above, so that 1/3 of workload can be removed for creators and they can produce more workflows. (edited)

Luke  [4:24 AM]

> In order to speed up the iteration process, the ideal case will actually be rebuilding the hub site from the repo
> What does this mean exactly? Revert back to using the templates repo as the source of truth rather than supabase hub backend?Jacob Segal  [5:13 AM]
> I think it might actually be more work to make the git repo the source of truth than it would be to finish the job and just make the Hub the source of truth. Remember that the hub has to support functionality (like automatic importing of models/assets) that simply doesn't exist in the json templates.
> How often do we actually change the spec for workflows or metadata?DrJKL (Alex)  [5:14 AM]
> Can we put it into the frontend repo?
> Jacob Segal  [5:14 AM]
> Put what? All the workflows on the hub?
> DrJKL (Alex)  [5:15 AM]
> The astro pieces
> Jacob Segal  [5:16 AM]
> Oh, I see. That definitely seems reasonable to me, but I think the original complaint was about the difficulty of publishing/updating the workflows themselves, right?
> DrJKL (Alex)  [5:17 AM]
> Yeah. We've had similar process discussions for the main website around model pages and marketing
> [5:17 AM]CC @bert @nav (edited) 
> Deep Mehta  [5:19 AM]
> Just want to add in — this was the first phase of discussion. We’re NOT starting on any action items right away.
> I’m trying to understand the pain points / prior decisions; and I’ve not looked at any actual piece of code yet either.

Before making any bigger changes, I’ll surface for the buy-in; at least for now, the paths why this was brought up:

I’m considering taking a look at what can we change for the in-app templates that allows for an “infinite workflow experience”
drift between hub / templates — this was for my understanding on why we have 2 separate source of truths
bottleneck on adding new workflows
Jacob Segal  [5:23 AM]
I think this is really just another place where we made a plan, did half of it, and then moved on to the next thing leaving things in a state that causes ongoing toil.
The original plan was to move to the hub as the source of truth with the ability to flag whether templates should show up in "templates" for local users. There are just a lot of features we want to support on the hub that aren't represented (and would be difficult to represent) in the template workflow json itself.Jo Zhang  [5:25 AM]
And we should bear in mind that the regular situation for both hub and template maintenance will be creatives without any dev support. So we don't want a situation where small changes will depend on cloud deploy or constant API changes.
I also explained the remaining work needed for the hub to be the source of truth to @Deep Mehta if we want to reduce the maintenance work and drift.
Either path we take, we should make sure it's fully resourced and implemented. (edited) 
Christian Byrne  [4:55 AM]
Not sure if mentioned but the biggest blocker to going fully to the database was making local reliant on pulling remote data and making network requests (for privacy reasons and bc it hurts offline usage). On privacy, I know we did already get alignment from all stakeholders. On offline usage, solution entertained was pulling templates in build process so there's an offline fallback. This would be a bit of a lift but not crazy (could be wrong). Jaewon had a PR to migrate all distributions to cloud templates API that was paused.

It would make things easier for frontend development if we moved to the hub API because then the hub app would not be coupled to workflow templates repo CI and we could move it into our monorepo (currently, website devs have to work cross repo). Of course, there are drawbacks (for those who maintain and edit templates) like losing everything that git provides w.r.t. version management, ci, change history, reviews, blame, agent accessibility, etc. (edited) 
Deep Mehta  [5:03 AM]
tbh, I was more curious on the other way around — why not make templates repo as the source of truth?
Anything we add to templates, can automatically go to everywhere ; and if we need workflows purely for hub, that can happen too.

Lmk if I’m missing something here
Christian Byrne  [5:07 AM]
Primary reason is likely what Jacob mentioned about wanting to associate templates with assets, have private templates, have more complex relations to other data/tables, and generally be more future proof for any product iteration we do. Additionally there are theoretical scaling issues with using git for big opaque data. That said, git argument is strong especially bc it's usable by agents
[5:07 AM]Overall just hard call to make and someone will be unhappy no matter what we do :sweat_smile:
