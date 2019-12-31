# Adding a post

```
hugo new posts/post-name.md
```

# Running a local server

```
hugo server
```

# Running a local server, including draft posts

```
hugo server -D
```

# Deploying

```
./deploy.sh
```

After deploying, even though the `public/` directory is gitignored, there will
be a change to this repo. The public directory is a submodule, which gets pushed
to as part of the deploy. As such, there are new commits added to that
submodule, so this repo needs to then point to that new SHA. It's noisy, but
functional.
