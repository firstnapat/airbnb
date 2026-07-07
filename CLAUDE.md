# CLAUDE.md

/## Orchestration workflow
You (Fable) are the orchestrator. Plan, decompose, synthesize.
Reasoning-heavy phases → deep-reasoner
Mechanical work → fast-worker
Codex (/codex:rescue --background) is a cracked engineer on par with deep-reasoner, from a different perspective. Treat as a peer, not a reviewer.
High-stakes decisions: task Opus + Codex on the same problem in parallel, synthesize the best of both, without showing either the other's answer. Keep your own context lean.

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

An Airbnb clone built with Ruby on Rails 8.1 (Ruby 3.4.9), PostgreSQL, Hotwire (Turbo + Stimulus), and Tailwind CSS 4. Authentication is handled by Devise.

## Commands

```sh
bin/setup              # Install deps, prepare DB (add --skip-server to not boot the app)
bin/dev                # Run the app: Rails server + Tailwind watcher (via Procfile.dev)
bin/rails db:prepare   # Create/migrate the database (PostgreSQL must be running; DB: airbnb_development)

bin/ci                 # Full CI pipeline: setup, rubocop, bundler-audit, importmap audit, brakeman
bin/rubocop            # Ruby style (rubocop-rails-omakase)
bin/brakeman           # Security static analysis
bin/importmap audit    # JS dependency vulnerability audit
```

There is no test suite yet (no `test/` or `spec/` directory) — `bin/ci` runs style and security checks only.

## Architecture

- **No JS bundler.** JavaScript uses importmap-rails: pin packages in `config/importmap.rb`, vendored libraries live in `vendor/javascript/` (e.g. `el-transition`, used for Tailwind-style enter/leave transitions in Stimulus controllers).
- **Stimulus controllers** in `app/javascript/controllers/` are auto-loaded via `pin_all_from`. Example: `header_controller.js` drives the header dropdown using `el-transition`.
- **Tailwind CSS 4** via tailwindcss-rails; styles live in `app/assets/tailwind/application.css`. The watcher runs as part of `bin/dev` — CSS changes won't appear if you run `bin/rails server` alone.
- **Devise** provides the `User` model (modules: database_authenticatable, registerable, recoverable, rememberable, validatable). Devise views are generated into `app/views/devise/` and are customized with Tailwind.
- **Layout structure:** `app/views/layouts/application.html.erb` renders the shared `_header.html.erb` partial. `app/views/tailwindcss_template/` holds reference HTML partials (navbar, sign-up) used as design templates — not routed views.
- **Routes:** `devise_for :users` plus `root "home#index"` (see `config/routes.rb`).
- **Solid trifecta** (solid_cache, solid_queue, solid_cable) is installed for caching, background jobs, and Action Cable; deployment is configured for Kamal + Thruster (see `Dockerfile`).
