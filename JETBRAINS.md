# JetBrains AI Assistant Guidelines

This file provides guidance to JetBrains AI Assistant (RubyMine, IntelliJ IDEA, etc.) when working with code in this repository.

## Quick Start Commands

### Development
- `bin/dev` - Start development server (Rails, Sidekiq, Tailwind)
- `bin/rails console` - Open Rails console
- `bin/setup` - Initial project setup

### Testing
- `bin/rails test` - Run all tests
- `bin/rails test test/models/account_test.rb` - Run specific test file
- `bin/rails test:system` - Run system tests (use sparingly)

### Code Quality
- `bin/rubocop -f github -a` - Ruby linting with auto-correct
- `bundle exec erb_lint ./app/**/*.erb -a` - ERB linting
- `bin/brakeman --no-pager` - Security analysis
- `npm run lint` - Check JavaScript/TypeScript
- `npm run format` - Format JavaScript/TypeScript

## Core Development Rules

### Authentication
- Use `Current.user` for current user (NOT `current_user`)
- Use `Current.family` for current family (NOT `current_family`)

### Architecture Pattern
- **Skinny Controllers, Fat Models** - Business logic in `app/models/`, not `app/services/`
- **Hotwire-First** - Use Turbo + Stimulus, prefer native HTML (`<dialog>`, `<details>`)
- **ViewComponents** over partials for complex/reusable UI elements

### Testing Philosophy
- Use Minitest + fixtures (NEVER RSpec or factories)
- Keep fixtures minimal (2-3 per model)
- Test critical code paths only
- Write tests as you go

## Internationalization (i18n)

### Critical Rule: All User-Facing Strings Must Be Localized

**Always use the `t()` helper for user-facing text:**

```erb
<!-- ✅ CORRECT -->
<h1><%= t("accounts.index.title") %></h1>
<%= link_to t("accounts.new"), new_account_path %>

<!-- ❌ WRONG -->
<h1>My Accounts</h1>
<%= link_to "New Account", new_account_path %>
```

### Community Localization Support

We welcome community translations! When adding a new language:

1. **Create locale directory**: `config/locales/[language_code]/` (e.g., `config/locales/fr/`)
2. **Mirror English structure**: Copy file organization from `config/locales/en/`
3. **Translate consistently**: Preserve interpolation variables (`%{name}`, `%{count}`)
4. **Update SUPPORTED_LOCALES**: Add language code to `app/helpers/languages_helper.rb`

**CRITICAL**: New languages must be added to `SUPPORTED_LOCALES` array in `app/helpers/languages_helper.rb`:

```ruby
# app/helpers/languages_helper.rb
SUPPORTED_LOCALES = [
  "en",   # English
  "de",   # German
  "es",   # Spanish
  "fr",   # French <- ADD NEW LANGUAGE HERE
  # ...
].freeze

# Also ensure it's in LANGUAGE_MAPPING
LANGUAGE_MAPPING = {
  # ...
  fr: "French",  # ADD IF MISSING
  # ...
}.freeze
```

Without updating `SUPPORTED_LOCALES`, the language will NOT appear in the UI!

### Translation Guidelines

- **Key Organization**: Hierarchical by feature: `accounts.index.title`, `transactions.form.amount_label`
- **Interpolation**: Preserve variables exactly: `%{name}`, `%{count}`, `%{amount}`
- **Pluralization**: Use Rails pluralization with `one`/`other` keys
- **Testing**: Switch language in settings and verify all major features
- **Quality**: Translate meaning in context, not just words

### Locale File Structure

```yaml
# config/locales/fr/accounts.yml
fr:
  accounts:
    index:
      title: "Mes comptes"
      new_account: "Nouveau compte"
    show:
      balance: "Solde"
      transactions_count:
        one: "%{count} transaction"
        other: "%{count} transactions"
```

## Styling with Tailwind

- Reference `app/assets/tailwind/maybe-design-system.css` for design tokens
- **Use functional tokens**: `text-primary` not `text-white`, `bg-container` not `bg-white`
- **Never create new styles** in design system files without permission
- **Generate semantic HTML** - prefer `<dialog>`, `<details>`, etc.

## Component Architecture

### When to Use ViewComponents

Use ViewComponents when:
- Element has complex logic or styling patterns
- Element will be reused across multiple contexts
- Element needs variants/sizes or interactive behavior
- Element requires accessibility features

### When to Use Partials

Use Partials when:
- Element is primarily static HTML
- Element is used in one or few specific contexts
- Element is simple template content
- Element doesn't need complex configuration

### Stimulus Controllers

**Use declarative approach** - HTML declares what happens:

```erb
<!-- ✅ GOOD: Declarative -->
<div data-controller="toggle">
  <button data-action="click->toggle#toggle" data-toggle-target="button">
    <%= t("components.toggle.show") %>
  </button>
  <div data-toggle-target="content" class="hidden">
    <p><%= t("components.toggle.content") %></p>
  </div>
</div>
```

- Keep controllers lightweight (< 7 targets)
- Component controllers stay in component directory
- Global controllers in `app/javascript/controllers/`
- Pass data via `data-*-value` attributes

## Database & Migrations

- Migrations inherit from `ActiveRecord::Migration[7.2]` (NOT 8.0)
- Simple validations (nulls, unique) in DB
- Complex validations in ActiveRecord
- Don't automatically run migrations

## Security Best Practices

- Never commit secrets (use `.env.local` from `.env.local.example`)
- Run `bin/brakeman` before major changes
- Use strong parameters in controllers
- Validate user input at boundaries

## Prohibited Actions

- Do NOT run `rails server` in responses
- Do NOT run `touch tmp/restart.txt`
- Do NOT run `rails credentials`
- Do NOT automatically run migrations

## Pre-Pull Request Checklist

Run these commands before creating a PR:

1. **Tests**: `bin/rails test` (always required)
2. **Linting**: `bin/rubocop -f github -a` and `bundle exec erb_lint ./app/**/*.erb -a`
3. **Security**: `bin/brakeman --no-pager`

Only proceed if ALL checks pass.

## Key Files & Directories

- `app/models/` - Business logic, domain models
- `app/controllers/` - Skinny controllers
- `app/components/` - ViewComponents for reusable UI
- `app/views/` - ERB templates
- `app/javascript/` - Stimulus controllers, JS
- `app/assets/tailwind/` - Design system, CSS
- `config/locales/` - i18n translations
- `test/` - Minitest tests
- `db/migrate/` - Database migrations

## Resources

- See [CLAUDE.md](CLAUDE.md) for comprehensive architecture details
- See [AGENTS.md](AGENTS.md) for general repository guidelines
- Rails i18n Guide: https://guides.rubyonrails.org/i18n.html
