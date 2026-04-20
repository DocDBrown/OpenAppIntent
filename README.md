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
assets
```

## Base Format

```
app-identity:
  name: ""
  purpose: ""
  validation:
    acceptance:
      - ""
      - ""
      - ""
user-workflows:
  - workflow: ""
    steps: [""]
  - workflow: ""
    steps: [""]
auth:
  tenancy_model: "single_tenant"
  authentication: ""
  authorization: ""
deployments:
  targets:
    - name: ""
      platform: ""
      language: ""
      framework: ""
      middleware: []
      deployment: ""
  external_services:
    service:
      - name: ""
        platform: ""
        type: ""
        type_name_ref: ""
secrets_provider: ""
domain-entities-and-relations: []
integrations: []
storage: []
flows: []
constraints: []
design: []
assets: []
```

## Example

```
app_identity:
  name: "recipe-journal"
  purpose: "A personal recipe journal where home cooks can save recipes, tag them by cuisine, and share read-only links with friends. Frontend is a React SPA; backend is a Go REST API."
  validation:
    acceptance:
      - "User can create a recipe with title, ingredients list, and instructions; POST /api/recipes returns 201 with {\"id\":\"<uuid>\",\"status\":\"created\"}"
      - "User can list their own recipes; GET /api/recipes returns 200 with {\"recipes\":[...],\"total\":<int>}"
      - "User can fetch a shared recipe by share_token; GET /api/shared/{token} returns 200 with the recipe JSON or 404"
      - "Frontend recipe list page renders within 1500ms on a 3G connection"
      - "All pages meet WCAG 2.1 AA contrast ratio requirements"
user-workflows:
  - workflow: "save recipe"
    steps:
      - "User opens the recipe editor in the frontend"
      - "User fills in title, ingredients, and instructions then clicks Save"
      - "Frontend posts the recipe to the backend and receives a confirmation"
      - "Frontend shows a success toast and navigates to the recipe detail page"
  - workflow: "share recipe"
    steps:
      - "User opens a recipe they own and clicks Share"
      - "Frontend requests a share_token from the backend"
      - "Frontend displays the share URL for the user to copy"
auth:
  tenancy_model: "single_tenant"
  authentication: "session_cookie"
  authorization: "owner_only"
deployments:
  targets:
    - name: "recipe-web"
      platform: "web_frontend_app"
      language: "react-vite"
      framework: "react"
      middleware: []
      deployment: "gitea.bossintent.com.au"
    - name: "recipe-api"
      platform: "backend_service"
      language: "go"
      framework: "net/http"
      middleware: []
      deployment: "gitea.bossintent.com.au"
external_services:
  service: []
secrets_provider: "env_var"
domain-entities-and-relations:
  - "entity:Recipe"
  - "attr:Recipe.id:uuid"
  - "attr:Recipe.title:string"
  - "attr:Recipe.ingredients:text"
  - "attr:Recipe.instructions:text"
  - "attr:Recipe.cuisine_tag:string"
  - "attr:Recipe.share_token:string"
  - "attr:Recipe.created_at:timestamp"
  - "rel:Recipe.belongs_to:User"
  - "entity:User"
  - "attr:User.id:uuid"
  - "attr:User.email:string"
  - "rel:User.has_many:Recipe"
integrations:
  - "from:recipe-web|to:recipe-api|type:sync|protocol:rest|direction:request-response|endpoint:https://api.recipe-journal.local|endpoint_env:RECIPE_API_URL|spec_ref:recipe-api.v1"
storage:
  - "store:recipes-db|type:postgresql|endpoint:postgres://recipes-db:5432/recipes|endpoint_env:DATABASE_URL|purpose:Recipes and users|entities:Recipe,User|owned_by:recipe-api|read_by:recipe-api|write_by:recipe-api"
flows:
  - "flow:create-recipe|trigger:user-action|step:Frontend validates form fields client-side|step:Frontend sends POST /api/recipes with session cookie|step:Backend persists Recipe row|step:Backend returns {\"id\":\"<uuid>\",\"status\":\"created\"}"
  - "flow:share-recipe|trigger:user-action|step:Frontend sends POST /api/recipes/{id}/share|step:Backend generates share_token and updates the Recipe row|step:Backend returns {\"share_url\":\"https://recipe-journal.local/shared/<token>\"}"
constraints:
  - "All API responses must use application/json content-type"
  - "Session cookies must be Secure, HttpOnly, SameSite=Lax"
  - "Frontend bundle size must not exceed 250KB gzipped"
  - "Recipe list page must render within 1500ms on a 3G connection"
design:
  - "Primary color: #2E7D5B (forest green)"
  - "Secondary color: #F5E6D3 (warm cream)"
  - "Typography: Inter for UI text, Playfair Display for recipe titles"
  - "Spacing scale: 4px base unit (4, 8, 16, 24, 32, 48, 64)"
  - "Border radius: 8px for cards, 4px for buttons and inputs"
  - "Dark mode support with system preference detection"
assets:
  - "asset:logo|path:public/logo.svg|kind:svg|serves:recipe-web"
  - "asset:favicon|path:public/favicon.ico|kind:icon|serves:recipe-web"
  - "asset:hero-image|path:public/images/hero.webp|kind:image|serves:recipe-web"
  - "asset:inter-font|path:public/fonts/Inter-Variable.woff2|kind:font|serves:recipe-web"
  - "asset:playfair-font|path:public/fonts/PlayfairDisplay-Variable.woff2|kind:font|serves:recipe-web"
```
