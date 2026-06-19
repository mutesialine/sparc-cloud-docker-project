# Docker Challenge: Dockerize the Todo App

**Objective:** Containerize a full-stack application by writing a multi-stage `Dockerfile` and a `compose.yaml` from scratch, combining a React frontend, a Node.js backend, and a MySQL database into a working multi-container setup with proper networking, data persistence, and live code syncing.

---

## Learning Objectives

By the end of this challenge you should be able to:

- Write a `Dockerfile` for a Node.js application, including installing dependencies and setting a start command
- Use multi-stage builds to produce separate development and production images from a single `Dockerfile`
- Explain why layer ordering matters for Docker build cache
- Copy build artifacts between stages using `COPY --from`
- Use Docker Compose to define and run a multi-container application
- Pass configuration to containers using environment variables
- Use health checks and `depends_on` to control service startup order
- Persist data across container restarts using named volumes
- Set up live code syncing between your local machine and a running container

---

> 💡 **Before you start:** Read the `README.md` to understand the app, then explore the codebase. The solution files already exist in this repo — only look at them once you've given it a real try.

---

## Part 1: The Dockerfile

### Instructions

1. **Containerize the backend**
   - Write a `Dockerfile` that builds and runs the Node.js backend.
   - The server should start and be reachable on port `3000`.

   **Verify:** Build the image and hit `http://localhost:3000/api/items` — you should get a JSON response.

2. **Handle the project's npm security setting**
   - Check the `.npmrc` file at the root of the project. Understand what it does and why it might cause a problem during the build.
   - Make sure your Dockerfile still produces a working image despite this setting.

   **Verify:** The backend starts without any dependency errors in the logs.

3. **Separate development and production builds**
   - Restructure your Dockerfile so you have a dedicated development image and a lean production image.
   - The production image should only contain what's needed to run the app — not dev tools or test files.

   **Verify:** Build both images and compare their sizes with `docker images`.

4. **Bundle the frontend into the production image**
   - Extend your Dockerfile to also build the React frontend.
   - The compiled frontend assets should end up being served by the backend in the production image.

   **Verify:** Run the production image and open `http://localhost:3000` — you should see the full todo app UI, not just the API.

5. **Make tests a required step before production**
   - The backend has a test suite. Add a stage that runs the tests.
   - Wire it up so the production image cannot be built if the tests fail.

   **Verify:** Break a test intentionally — the production build should fail. Fix it, and it should pass again.

---

## Part 2: The Compose File

### Instructions

1. **Define your services**
   - Write a `compose.yaml` that runs the backend and connects it to a MySQL database.
   - The backend needs database credentials passed in as environment variables — check `backend/src/persistence/index.js` to see what it expects.

   **Verify:** Run `docker compose up` and confirm the backend connects to MySQL successfully.

2. **Make the database reliable**
   - MySQL takes a few seconds to be ready after the container starts. Make sure the backend doesn't crash while waiting for it.
   - Persist the database data so it survives a `docker compose down` and back up.

   **Verify:** Add a todo item, run `docker compose down`, then `docker compose up` again — your item should still be there.

3. **Add the frontend as a service**
   - Add the React dev server as a service.
   - Set up live code syncing so that changes to source files are reflected in the running containers without a full rebuild.

   **Verify:** Run `docker compose up --watch`, edit a file in `client/src/`, and confirm the change appears in the browser automatically.

4. **Route frontend and backend through a single port**
   - In development, the frontend runs on port `5173` and the backend on port `3000`. Add a reverse proxy (Traefik) so both are accessible from port `80`, with requests to `/api/*` going to the backend and everything else going to the frontend.

   > 💡 Don't spend too long on this one if you're unfamiliar with Traefik — check the docs or ask a coach.

   **Verify:**
   - `http://localhost` → React app
   - `http://localhost/api/items` → JSON from the backend

5. **Add a database UI**
   - Add phpMyAdmin as a service so you can inspect the database through a browser.
   - It should be accessible at `http://db.localhost` (not `http://localhost`).

   **Verify:** Open `http://db.localhost` and confirm you can see the `todos` database and its data.

---

## Final Check

```bash
# Development stack with live reload
docker compose up --watch
```

- `http://localhost` → todo app
- `http://db.localhost` → phpMyAdmin

```bash
# Production build
docker build --target final -t todo-app-prod .
docker run -p 3000:3000 todo-app-prod
```

- `http://localhost:3000` → fully bundled app, frontend and backend together

Once both work, compare your files to the solution already in this repo and see how your approach differs.

---

