# Contract Kit - Setup, Build, and Usage Guide

Contract Kit is a Next.js web application for building API contracts, generating OpenAPI/Java artifacts, and publishing OpenAPI YAML files to GitHub Releases.

## 1. What This Project Does

- Build API contracts from scratch in a UI.
- Import existing Swagger/OpenAPI specs from URL.
- Generate:
  - OpenAPI YAML
  - Java Spring controller + DTO artifacts
  - (From API Generator flow) additional generated assets bundled for download
- Publish generated YAML to GitHub Releases.
- Keep local history and notification state in browser local storage.

## 2. Tech Stack

- Next.js 14
- React 18
- TypeScript
- Tailwind CSS

## 3. Prerequisites

Install the following on your machine/server:

- Node.js 18.x or 20.x (LTS recommended)
- npm (comes with Node)
- Git

Check versions:

```bash
node -v
npm -v
git --version
```

## 4. Project Setup (Local Development)

From the project root:

```bash
npm install
npm run dev
```

Open in browser:

- `http://localhost:3000`

### Available Scripts

- `npm run dev` - start development server
- `npm run build` - create production build
- `npm run start` - run production server (after build)
- `npm run lint` - run Next.js lint checks
- `npm run openapi:gen:frontend` - generate Angular TypeScript client from OpenAPI spec
- `npm run openapi:gen:backend` - generate Java Spring server stubs from OpenAPI spec
- `npm run openapi:gen:all` - run both frontend and backend OpenAPI generation

## 5. Environment Variables

This project currently does not require mandatory `.env` variables to run.

Important notes:

- GitHub Personal Access Token (PAT) is entered in the UI when publishing.
- PAT is used from the browser flow; do not share tokens and use least-privilege scopes.
- Recommended token permissions for release publishing:
  - `Contents: Read and write`

## 6. Build and Run on Server (Production)

## Option A: Direct Node process

```bash
# 1) Clone and enter repo
git clone <your-repo-url>
cd contractkit

# 2) Install dependencies
npm install

# 3) Build app
npm run build

# 4) Start production server (default port 3000)
npm run start
```

The app serves on:

- `http://<server-ip>:3000`

To run on custom port:

```bash
PORT=4000 npm run start
```

## Option B: Run with PM2 (recommended for persistence)

```bash
npm install -g pm2
pm2 start npm --name contractkit -- start
pm2 save
pm2 startup
```

Useful PM2 commands:

```bash
pm2 status
pm2 logs contractkit
pm2 restart contractkit
```

## 7. Reverse Proxy (Recommended)

Use Nginx/Apache in front of the Node process for HTTPS and domain routing.

Nginx upstream target:

- `127.0.0.1:3000` (or your configured PORT)

Make sure proxy headers/WebSocket forwarding are enabled for Next.js.

## 8. How to Use the Product

## 8.1 Sign In Screen

- App starts with a UI login/register simulation.
- Enter name/email and continue with Google/GitHub button.
- This is local UI auth flow (no real OAuth backend yet).

## 8.2 Create Project (Builder)

1. Open `Create Project`.
2. Fill project setup:
   - Project Name
   - Module Name
   - API Version
   - Base Package
   - Base Request Path
   - Controller Name
   - Service Name
3. Add endpoints and configure:
   - HTTP method and path
   - Request fields
   - Response fields
   - Annotation options (`@PreAuthorize`, `@Loggable`, `@JsonView`)
4. Use live YAML preview.
5. Download YAML and generated code artifacts.

## 8.3 API Contract Builder (Importer Flow)

1. Open `API Contract Builder`.
2. Provide OpenAPI URL and click load.
   - The app fetches URL through `/api/openapi-proxy` to reduce CORS issues.
3. Edit grouped controllers/endpoints.
4. Save changes.
5. Export JSON/YAML.

## 8.4 Publish YAML to GitHub Release

In publish panel, provide:

- GitHub owner (user/org)
- Repository
- Tag (example: `v1.0.0`)
- Asset file name (example: `openapi.yaml`)
- GitHub PAT

Then click publish.

What happens:

- Checks or creates release by tag.
- Uploads YAML as a release asset.
- Replaces existing asset with same name when applicable.

## 8.5 History and Notifications

- Published items and notifications are stored in browser local storage.
- Data is browser/profile specific.
- Clearing browser storage clears local app history.

## 9. Security and Operational Guidance

- Use organization-scoped PAT with minimum required permissions.
- Rotate PAT periodically.
- Prefer HTTPS in production.
- If exposing publicly, restrict outbound access if needed because proxy endpoint can fetch remote URLs.

## 10. Troubleshooting

## Build fails

- Reinstall dependencies:

```bash
rm -rf node_modules package-lock.json
npm install
```

- Re-run:

```bash
npm run build
```

## OpenAPI import fails

- Verify URL is reachable from server/browser.
- Ensure URL is valid `http` or `https`.
- Check API route errors in browser network tab.

## GitHub publish fails

- Confirm owner/repo/tag/asset/token are all provided.
- Confirm PAT has write permission to target repo.
- Confirm repo exists and account has access.
- Check tag conflicts/asset naming.

## 11. Recommended Team Workflow

1. Pull latest code.
2. Create feature branch.
3. Implement and test changes locally (`npm run dev`, `npm run lint`, `npm run build`).
4. Raise PR and review.
5. Merge and deploy using production build process.

## 12. Maintenance Checklist

- Keep Node.js LTS updated.
- Update dependencies regularly (`npm audit`, dependency updates).
- Validate release publishing flow after GitHub permission changes.
- Back up important generated artifacts (history in browser storage is not centralized).

## 13. OpenAPI Code Generation (UI Team + Backend Team)

This section is the standard reference for generating TypeScript (Angular) and Java (Spring) files from OpenAPI.

### 13.1 Install OpenAPI Generator CLI (npm)

```bash
npm install --save-dev @openapitools/openapi-generator-cli
```

### 13.2 Input Spec

Default spec path used by scripts:

- `./contracts/openapi.yaml`

Update this path in scripts/commands if your spec is in a different location.

### 13.3 Generate Frontend (Angular TypeScript)

```bash
npx @openapitools/openapi-generator-cli generate \
  -i ./contracts/openapi.yaml \
  -g typescript-angular \
  -o ./generated/frontend/angular-client \
  --additional-properties=npmName=@bahwan/contractkit-api,npmVersion=1.0.0,providedIn=root
```

### 13.4 Generate Backend (Java Spring)

```bash
npx @openapitools/openapi-generator-cli generate \
  -i ./contracts/openapi.yaml \
  -g spring \
  -o ./generated/backend/spring-server \
  --additional-properties=groupId=com.bahwan.contractkit,artifactId=contractkit-api,artifactVersion=1.0.0,useSpringBoot3=true,interfaceOnly=true
```

### 13.5 Run Both

```bash
npm run openapi:gen:all
```

Or directly:

```bash
npx @openapitools/openapi-generator-cli generate -i ./contracts/openapi.yaml -g typescript-angular -o ./generated/frontend/angular-client --additional-properties=npmName=@bahwan/contractkit-api,npmVersion=1.0.0,providedIn=root
npx @openapitools/openapi-generator-cli generate -i ./contracts/openapi.yaml -g spring -o ./generated/backend/spring-server --additional-properties=groupId=com.bahwan.contractkit,artifactId=contractkit-api,artifactVersion=1.0.0,useSpringBoot3=true,interfaceOnly=true
```

