# Organization setup

To run these workshops in a company context, we set up a GitHub Organization if one is not already present.

## What is an organization

A GitHub organization is a shared “container” where a team, company, or community keeps their repositories, people, and permissions together under a single name (for example, `github.com/company-name`). [docs.github](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/about-organizations)

### What it is in practice

- It is a type of account separate from personal accounts, with its own name, logo, and settings. [docs.github](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/about-organizations)
- Inside an organization, you can have many repositories, public or private, linked to the same group or company. [github](https://github.com/orgs/community/discussions/69020)
- Users join with their personal account but work “inside” the organization. [docs.github](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/about-organizations)

### What is it for (super concise)

- **Team management**: define roles (owner, maintainer, member), create teams with different permissions (read-only, write, admin) on groups of repos. [github](https://github.com/orgs/community/discussions/69020)
- Centralize projects: all company or product repos are in the same space, organized and easy to find (avoiding many repos scattered among personal accounts). [it.scribd](https://it.scribd.com/document/513270621/github-guide-to-organizations)
- Control access: when someone joins or leaves the team, you grant or revoke access once at the organization level, without managing individual invites on each repo. [github](https://github.com/orgs/community/discussions/69020)
- Give identity to the code: projects are released with the organization “brand” (useful for companies, OSS, communities). [docs.github](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/about-organizations)
- Use advanced tools: you can configure security policies, GitHub Actions, GitHub Projects, issues, common templates, etc. at the organization level. [nethesis](https://www.nethesis.it/approfondimenti/open-source/github-cos-e)

## Setup

1. Go to new org page to create a [new organization for free](https://github.com/account/organizations/new?plan=free).
1. Select "a business or institution"
1. Add-ons: activate Copilot Business in this organization

![organization-setup](image.png)
