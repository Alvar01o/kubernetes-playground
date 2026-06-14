# Dockerizing the Test App

Steps to build and run the `test-app` Docker image locally.

> **Prerequisite:** Docker is running and you are inside the `test-app` directory.
> ```bash
> cd test-app
> ```

---

## 1. Build the Image

Run the build from the directory that contains the `Dockerfile`:

```bash
docker build .
```

Tag the image with a name so you don't have to reference it by its ID later:

```bash
docker build -t test-app .
```

> `docker --build` is not a valid command — the correct form is `docker build`.

---

## 2. Verify the Image Was Created

```bash
docker images
```

You should see `test-app` listed with its tag, size, and creation time.

---

## 3. Run the Container

Map port 3000 on your machine to port 3000 inside the container:

```bash
docker run -p 3000:3000 test-app
```

Test it:

```bash
curl http://localhost:3000
# {"message":"testing kubernetes"}
```

---

## Troubleshooting: Port Already in Use

If Docker fails with `address already in use`, another process is holding port 3000.

**Find and kill it:**

```bash
# Find the process using port 3000
sudo lsof -i :3000

# Kill it using the PID shown in the output
sudo kill -9 <PID>
```

Then retry the `docker run` command.

Alternatively, map to a different local port to avoid the conflict entirely:

```bash
# Expose the app on localhost:3001 instead
docker run -p 3001:3000 test-app
```

---

## Useful Container Commands

```bash
# List running containers
docker ps

# Stop a running container
docker stop <container-id>

# Remove a stopped container
docker rm <container-id>

# Run in detached mode (background)
docker run -d -p 3000:3000 test-app
```
