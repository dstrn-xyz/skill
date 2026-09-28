<p align="center">
  <a href="https://framework.dstrn.xyz" target="_blank"><img src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.images/text_logo.png" width="400"></a>
</p>
<p align="center">
official agent engineering skill and architectural guidelines for dframework.
</p>

<p align="center">
  <a href="https://github.com/dstrn-xyz/framework/actions"><img src="https://github.com/dstrn-xyz/framework/actions/workflows/tests.yml/badge.svg" alt="build status"></a>
  <img src="https://img.shields.io/badge/version-0.21.0-d3ac5f" alt="version">
  <a href="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/LICENSE"><img src="https://img.shields.io/badge/license-apache%202.0-blue.svg" alt="license"></a>
  <img src="https://img.shields.io/badge/established_in-2019-d3ac5f" alt="established in 2019">
  <a href="https://dstrn.xyz"><img src="https://img.shields.io/badge/copyright-dstrn-d46a6a" alt="copyright"></a>
</p>

<a name="about-dframework-skill"></a>

## about dframework skill

official agent engineering skill providing architectural guidelines, conventions, rules, and reference documentation for ai coding assistants working with dframework applications.

<a name="structure"></a>

# structure

the skill is organized into a core mandate file and detailed topic references.

| file                                           | description                                                                              |
| ---------------------------------------------- | ---------------------------------------------------------------------------------------- |
| `SKILL.md`                                     | core engineering mandate, fundamental conventions, and verification checklist            |
| `references/architecture-and-anti-patterns.md` | project directory layout, architectural boundaries, and common anti patterns             |
| `references/cli-commands.md`                   | command line interface reference for lifecycle, generators, database, and native builds  |
| `references/controllers-routing-responses.md`  | routing definitions, controller actions, input validation, and response helpers          |
| `references/frontend-components.md`            | catalog of built in ui custom elements and interactive components                        |
| `references/frontend-design-system.md`         | three tier surface elevation, typography hierarchy, micro gaps, and view compute         |
| `references/frontend-spa-reactivity.md`        | utility classes, client navigation router, and reactivity directives                     |
| `references/globals-facades-helpers.md`        | global facades, response helpers, and explicit framework imports                         |
| `references/jobs-scheduler-websockets.md`      | background worker jobs, task scheduler frequencies, and websocket routing                |
| `references/localization-and-i18n.md`          | locale facade, translation directives, furigana ruby annotations, and dictionary linting |
| `references/models-database-migrations.md`     | active record models, relationships, query builder, and database migrations              |
| `references/native-plugins-bridge.md`          | native bridge apis, four file zero config plugins, and platform simulators               |
| `references/spa-router.md`                     | client side routing lifecycle, link interception, and morph transitions                  |
| `references/views-templates.md`                | template directives, layout inheritance, section slots, and server javascript blocks     |

<a name="license"></a>

# license

[apache 2.0](https://github.com/dstrn-xyz/docs/blob/main/LICENSE)