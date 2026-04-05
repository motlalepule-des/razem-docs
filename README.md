# Razem Documentation

Official documentation for the **Razem** financial services platform by Digital Edge Solutions (DES).

## Live Site

The documentation is hosted on GitHub Pages:  
**[https://motlalepule-des.github.io/razem-docs](https://motlalepule-des.github.io/razem-docs)**

## Documentation Structure

```
razem-docs/
├── index.md                    # Home page
├── _config.yml                 # Jekyll configuration (Just the Docs theme)
├── Gemfile                     # Ruby dependencies
├── .github/
│   └── workflows/
│       └── pages.yml           # GitHub Pages deployment workflow
└── docs/
    ├── guides/
    │   ├── index.md            # Guides overview
    │   ├── getting-started.md  # Getting started guide
    │   ├── examples.md         # Code examples
    │   └── webhooks.md         # Webhooks integration guide
    ├── api/
    │   ├── index.md            # API reference overview
    │   ├── authentication.md   # Authentication guide
    │   ├── accounts.md         # Accounts API
    │   ├── payments.md         # Payments & Transactions API
    │   └── beneficiaries.md    # Beneficiaries API
    ├── account/
    │   ├── index.md            # My Razem Account overview
    │   ├── setup.md            # Account setup & KYC
    │   ├── payments.md         # Managing payments (portal)
    │   └── security.md         # Account security
    ├── architecture/
    │   ├── index.md            # Architecture overview
    │   ├── overview.md         # System architecture
    │   └── deployment.md       # Deployment guide
    └── reference/
        ├── index.md            # Reference overview
        ├── errors.md           # Error codes reference
        └── changelog.md        # API changelog
```

## Running Locally

```bash
# Install dependencies
bundle install

# Serve locally with live reload
bundle exec jekyll serve --livereload

# Build for production
bundle exec jekyll build
```

Visit `http://localhost:4000/razem-docs` to view the site.

## Contributing

1. Create a feature branch from `main`
2. Make your changes to the markdown files in `docs/`
3. Test locally with `bundle exec jekyll serve`
4. Open a pull request

## Tech Stack

- **Static Site Generator:** [Jekyll](https://jekyllrb.com/)
- **Theme:** [Just the Docs](https://just-the-docs.com/)
- **Hosting:** [GitHub Pages](https://pages.github.com/)

---

&copy; 2025 Digital Edge Solutions. All rights reserved.
