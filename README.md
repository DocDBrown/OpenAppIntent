# OpenAppIntent Format
- Intent Driven Devlopment Yaml Spec
- For Application Level Intent Driven Devlopment 
  
## Mandatory Fields

```
app_identity
user_workflows
auth
deployments.targets
```

## Optional Fields

```
deployments.external_services
secrets_provider
domain_entities_and_relations
integrations
storage
flows
constraints
design
```

## Example

```
app_identity:
  name: "team-standup-bot"
  purpose: "Collect daily standup updates from team members via Slack, store them, and post a summary to a designated channel at a configured time each day."
    validation:
    acceptance:
      - "Team member can submit standup update via Slack slash command with yesterday/today/blockers fields"
      - "Bot posts formatted summary of all submissions to the configured channel at 9:30am"
      - "Team lead can view historical standups for any team member for the past 30 days"
      - "Admin can configure which channel receives the summary and what time it posts"
user_workflows:
  - workflow: "Submit daily standup"
    steps:
      - "Team member types /standup in any Slack channel"
      - "Bot opens a modal with fields for yesterday, today, and blockers"
      - "Team member fills in fields and submits"
      - "Bot confirms submission with an ephemeral message"
  - workflow: "View standup history"
    steps:
      - "Team lead types /standup-history @username"
      - "Bot responds with the last 5 standups from that user in an ephemeral message"
      - "Team lead can page through older entries"
auth:
  tenancy_model: "multi-tenant"
  authentication: "oauth2-oidc"
  authorization: "rbac"
deployments:
  targets:
    - name: "standup-api"
      platform: "fly-io"
      language: "typescript"
      framework: "fastify"
  external_services:
    service:
      - name: "slack"
        platform: "slack"
        type: "messaging"
        type_name_ref: "slack-bolt-v3"
secrets_provider: "env-var"
domain_entities_and_relations:
  - "entity:StandupEntry"
  - "attr:StandupEntry.yesterday:text"
  - "attr:StandupEntry.today:text"
  - "attr:StandupEntry.blockers:text"
  - "attr:StandupEntry.submitted_at:timestamp"
  - "rel:StandupEntry.belongs_to:User"
  - "rel:StandupEntry.belongs_to:Team"
  - "entity:Team"
  - "attr:Team.slack_channel_id:string"
  - "attr:Team.summary_post_time:time"
  - "rel:Team.has_many:User"
  - "rel:Team.has_many:StandupEntry"
  - "entity:User"
  - "attr:User.slack_user_id:string"
  - "attr:User.display_name:string"
  - "attr:User.role:enum(member,lead,admin)"
  - "rel:User.belongs_to:Team"
  - "rel:User.has_many:StandupEntry"
storage:
  - "store:primary-db|type:postgresql|endpoint:postgres://primary-db.default.svc.cluster.local:5432/standups|endpoint_env:DATABASE_URL|purpose:Standup entries, teams, and users|entities:StandupEntry,Team,User|owned_by:standup-api|read_by:standup-api|write_by:standup-api"
integrations:
  - "from:standup-api|to:slack|type:sync|protocol:rest|direction:request-response|endpoint:https://slack.com/api|endpoint_env:SLACK_API_URL"
  - "from:standup-api|to:keycloak|type:sync|protocol:oidc|direction:request-response|endpoint:http://keycloak.keycloak.svc.cluster.local:8080|endpoint_env:KEYCLOAK_BASE_URL"
flows:
  - "flow:daily-summary-post|trigger:cron:team.summary_post_time|step:Query all standup entries for today for the team|step:Format entries into Slack block kit message|step:Post summary to team.slack_channel_id"

```
