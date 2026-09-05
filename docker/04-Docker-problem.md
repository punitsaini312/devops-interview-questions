**The Problem we solved in my last company**

One Docker-related problem I faced was that one of our microservices was taking a long time to build, and the final image was also bigger than we wanted.

We were building the image through GitHub Actions. Since our CI jobs were running on fresh machines, we couldn't rely on the normal Docker build cache being available from the previous build.

The main reason the build was taking time was that we had a lot of Python dependencies and some OS-level dependencies that didn't change very often, but they were being installed again during every build.

So instead of doing everything in one Dockerfile, we separated the stable dependencies from the application code.

We created a separate `Dockerfile.base` that installed the Python and OS dependencies and built a base image. We pushed that image to our container registry.

Then our main Dockerfile simply started from that base image:

```dockerfile
FROM .../coreai-base:v5

WORKDIR /app

COPY app/ ./app/

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Now, during the normal application build, we weren't reinstalling all those dependencies. We mainly had to copy the application code on top of the already-prepared base image.

This significantly reduced our build time and also helped us keep the final application image cleaner because the heavy build tools stayed in the builder stage and weren't included in the runtime image.

So the main idea was to **separate things that change rarely from things that change frequently**.


If they ask: "Why did you create two Dockerfiles?"

You can answer naturally:

"Because our dependencies didn't change frequently, but our application code changed frequently. We didn't want to rebuild all the dependencies every time we changed the application. So we created a reusable base image containing the dependencies and used that as the starting point for the application image."

And if they ask:

"Why not just use Docker layer caching?"

Say:

"We were building in GitHub Actions on fresh machines, so the local Docker cache wasn't reliably available between builds. We could have configured a remote Docker cache, but for our case we chose a reusable base image because the dependency layer was relatively stable and could be built separately."

That last answer is particularly good because it shows you understand that a base image isn't the only possible solution.
