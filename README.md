# Cloning

The theme is included as a submodule, so after cloning, you must set up that
submodule:

```
git submodule init
git submodule update
```

# Updating the theme

The theme is a submodule, so to get the latest changes from the remote, you can
execute:

```
cd themes/hugo-kiera
git fetch
git merge origin/master
cd ../..
```

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

Currently, this is hosted on Github pages. Hugo's documentation provides
[instructions](https://gohugo.io/hosting-and-deployment/hosting-on-github/), which at this point means only this script needs to run to deploy:

```
./deploy.sh
```

After deploying, even though the `public/` directory is gitignored, there will
be a change to this repo. The public directory is a submodule, which gets pushed
to as part of the deploy. As such, there are new commits added to that
submodule, so this repo needs to then point to that new SHA. It's noisy, but
functional.
