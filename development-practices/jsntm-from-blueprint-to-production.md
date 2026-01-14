# JSNTM: From MFE Blueprint to Development Platform in 5 Days

![JSNTM - Just Say No to Monoliths](../images/jsntm.png)

This is Part 3 of the JSNTM series. [Part 1](https://www.nathanfox.net/p/jsntm-just-say-no-to-monoliths-in) made the pledge to stop creating monoliths. [Part 2](https://www.nathanfox.net/p/jsntm-micro-frontends-svelte) presented a reference implementation for micro-frontends. This post covers what happened when I applied those patterns at work.

## The Timeline

**Days 1-2:** MFE Shell with authentication, authorization, and full CI/CD deployed to the development Kubernetes cluster.

**Days 3-5:** First MFE with basic functionality, integrated with its API and deployed alongside the shell.

Five days from concept to working code on the development platform. Not a proof of concept—real infrastructure with real authentication hitting real APIs.

This is what happens when GenAI-assisted development meets [Example-Driven Development](https://www.nathanfox.net/p/example-driven-development-using-ai-agent-claude-code).

## The Setup

The existing system runs on Kubernetes with an API gateway handling path-based routing. Multiple services serve different bounded contexts, each accessible via `/api/service-name/`.

The frontend, however, is a single Vue application. Every feature, every team, one monolithic SPA. Classic frontend monolith hiding behind backend services.

Time to apply the JSNTM pledge to the UI layer.

## Days 1-2: The Shell

The MFE Shell needed to:
- Integrate with existing Azure AD authentication (already used by the Vue app)
- Provide navigation, theming, and shared utilities
- Load MFEs dynamically based on a manifest
- Deploy to the same Kubernetes cluster via existing CI/CD patterns
- Work alongside the existing Vue application (gradual migration, not replacement)

I used [Planning-Driven Development](https://www.nathanfox.net/p/planning-driven-development) to create the planning documents first—a 25-page planning document and an 8-page implementation plan. The [reference implementation](https://github.com/nathanfox/mfe-svelte-shell-example) served as the example—my GenAI agent analyzed its patterns and adapted them for the production environment.

The planning document captured the specifics:
- The existing app uses MSAL for Azure AD—integrate with the same auth library
- Follow the existing Helm chart patterns for Kubernetes deployment
- Configure API gateway mappings for `/mfe/` routes

With the plan in place, implementation was methodical. Each phase had clear deliverables. By end of day 2, the shell was deployed and accessible, showing a working sidebar and authentication flow.

## The First Lesson: Path Conflicts

Our API gateway routes requests based on path prefixes. We needed:
- `/mfe/` - The shell application
- `/mfes/` - Individual MFE bundles

That single character difference (`/mfe/` vs `/mfes/`) matters.

Initially, both the shell SPA routes and MFE bundles lived under `/mfe/`. The gateway couldn't distinguish between a shell page request (`/mfe/feature-x`) and an MFE bundle request (`/mfe/feature-x/remoteEntry.js`). Moving MFE bundles to `/mfes/` gave each concern its own routing prefix.

The shell serves its SPA from `/mfe/`. MFE bundles load from `/mfes/{mfe-name}/remoteEntry.js`. Two distinct routing concerns, two distinct paths.

## Days 3-5: The First MFE

With the shell deployed, the next step was proving the architecture with a real MFE. A new feature needed a UI for file uploads, processing status, and search functionality—a perfect candidate for a self-contained module.

The MFE needed to:
- Implement the lifecycle contract (bootstrap, mount, unmount)
- Communicate with its own API endpoints
- Register navigation routes with the shell
- Deploy independently from the shell

Here's where it got interesting.

## The Revelation: MFE Inside the Microservice

The conventional approach: create a separate repository for the MFE. Separate CI/CD. Separate versioning.

Instead, we put the MFE directory inside the microservice's API repository:

```
microservice-api/
├── src/
│   └── api/           # API source code
├── tests/
├── mfe/               # MFE lives here
│   ├── src/
│   ├── Dockerfile
│   └── package.json
├── helm/
│   ├── api/           # API Helm chart
│   └── mfe/           # MFE Helm chart
└── azure-pipelines.yml
```

One repository. One CI/CD pipeline. One team.

When the API changes, the MFE updates in the same commit. When the MFE ships, the API ships with it. Version compatibility is guaranteed by colocation.

This is true vertical slicing—from database to API to UI, all owned by the same bounded context.

## Why This Works So Well

**Single Source of Truth:** API types and MFE types stay synchronized. When an API response changes, the MFE types update in the same PR.

**Simplified Code Review:** Reviewers see the full feature—API changes and UI changes together. No cross-repository coordination.

**Unified CI/CD:** One pipeline builds both artifacts. The API container and MFE container deploy as a unit when appropriate, or independently when needed.

**GenAI Context:** Your coding agent sees both the API and MFE in the same workspace. "Add a new field to this API response and display it in the UI" becomes a single conversation, not coordination across repositories.

**Team Ownership:** The team owns the bounded context completely. No handoffs between "backend team" and "frontend team" for feature development.

## The Authentication Story

The existing Vue application already used MSAL for Azure AD authentication. Users were already logged in. How does the shell access that authentication state?

The answer: shared token cache.

MSAL stores tokens in `localStorage` by default. The Vue app and the MFE Shell run on the same domain. Same domain means same `localStorage`. Same `localStorage` means the shell can access tokens the Vue app already acquired.

No duplicate login prompts. The user logs into the Vue app once, navigates to an MFE route, and they're already authenticated. The shell's auth store initializes from the shared cache.

This works because:
- Same domain (no cross-origin issues)
- Same Azure AD tenant and client configuration
- Same MSAL library storing tokens with the same cache keys

For greenfield deployments, the shell would handle initial authentication. In migration scenarios like this, the shared cache provides seamless SSO.

## Static Deployment with Nginx

Both the shell and MFEs deploy as static files served by Nginx containers. No Node.js runtime in production. No server-side rendering.

The Dockerfile follows a multi-stage pattern:
1. Node.js build stage: `npm run build`
2. Nginx production stage: Copy built files, apply nginx.conf

The nginx configuration handles:
- SPA routing (all paths serve index.html)
- Aggressive caching for hashed assets (1 year, immutable)
- No caching for entry points (index.html, remoteEntry.js)
- Security headers
- Gzip compression

## The Manifest

MFEs register via a JSON manifest that the shell loads at startup:

```json
{
  "mfes": [
    {
      "id": "feature-x",
      "name": "Feature X",
      "entry": "/mfes/feature-x/remoteEntry.js",
      "route": "/feature-x",
      "requiredRoles": ["RoleA", "RoleB"],
      "menu": {
        "label": "Feature X",
        "icon": "FileText",
        "order": 1
      }
    }
  ]
}
```

Adding a new MFE means adding an entry to the manifest and deploying the bundle. The shell discovers and loads it automatically.

Role-based filtering happens client-side. If the user lacks required roles, the menu item doesn't appear. This complements API-level authorization—the backend still validates permissions on every request.

## Lessons for Others

**Separate shell and MFE paths.** `/mfe/` for the shell, `/mfes/` for bundles. Avoid routing conflicts in your API gateway.

**Colocate MFEs with their APIs.** Put the MFE inside the microservice repository. Vertical slices simplify everything.

**Leverage existing auth infrastructure.** If you have working authentication, share the token cache. Don't duplicate login flows.

**Static deployment is enough.** Nginx serving built files is simple, fast, and sufficient for SPAs and MFEs.

**Start with one MFE.** Prove the architecture with a single module before migrating existing features.

## What's Next

The shell is deployed. The first MFE is live on the development platform. The pattern is proven.

Future MFEs follow the same template. Clone the MFE directory structure, implement the lifecycle contract, add a manifest entry. GenAI accelerates each iteration using the first MFE as the example.

The existing Vue monolith doesn't need to migrate all at once. New features can be MFEs from day one. Existing features can migrate as capacity allows. The shell and Vue app coexist, sharing authentication and routing.

This is incremental architecture improvement—enabled by GenAI development speed and [Example-Driven Development](https://www.nathanfox.net/p/example-driven-development-using-ai-agent-claude-code).

## The JSNTM Pledge Continues

Part 1: No more monolithic backend services.
Part 2: A reference implementation for the frontend.
Part 3: Validation on the development platform in 5 days.

The patterns work. The timeline is real. The only barrier is starting.

---

*This post is part of the JSNTM series on eliminating monoliths with GenAI-assisted development. See [Part 1](https://www.nathanfox.net/p/jsntm-just-say-no-to-monoliths-in) for the original pledge and [Part 2](https://www.nathanfox.net/p/jsntm-micro-frontends-svelte) for the reference implementation.*

---

*This post was written with a GenAI coding agent. I described the experience and the agent helped identify patterns and structure the content. I reviewed and edited the result. This is how I work now. You can see the revision history in my [blog posts repo](https://github.com/nathanfox/nathan-fox-net-posts).*
