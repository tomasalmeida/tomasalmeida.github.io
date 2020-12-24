---
title: "Creating a blog using Hugo, travis-ci and Github pages"
date: 2020-12-24T12:58:00+01:00
draft: false
---
# How I created this blog 

As many github blogs born and die, my first post is how I created this blog using hugo as static page generator, travis-ci as CI/CD and github pages as page hosting.

As I do not want to have 2 separate repositories as proposed by Claudio Jolowicz [in his tutorial](https://cjolowicz.github.io/posts/hosting-a-hugo-blog-on-github-pages-with-travis-ci/), I will use the possibility to host the source under `main` and the page itself under another branch. Github proposes as a standard to use `gh-pages`for the content itself, so I will follow it.


## Github configuration
### Create your github repository

Under your github account, create a new repository using the format `<USERNAME>.github.io` (in my case it is `tomasalmeida.github.io`). Pay attention the repository is public, I tried it as private and it did not work, so if it works for you, even better :-)

[image 1]

### Set gh-pages as content branch
Now, let's create our needed branch `gh-pages` and set it as the default content container. To do it, we need to start the repo locally and push the changes. I made a dummy commit, but you can directly create your hugo content if you prefer...

#### Create the repo locally

We will create the repository locally and create some content
```shell
mkdir tomasalmeida.github.io
cd $_
echo "# Visit https://tomasalmeida.github.io" >> README.md
git init
git add README.md
git commit -m "Here a new blog is born"
```

#### Create the needed branches
```shell
git branch -M main # I follow github naming and I use main instead of master
git branch gh-pages # the content branch
```

#### Link local and remote repository and push everything
```shell
git remote add origin git@github.com:tomasalmeida/tomasalmeida.github.io.git
git push origin '*:*'
git push origin --all # just a double check
```

#### Change the github content branch

Go to your github repository again and enter in settings. There scroll down to **GitHub Pages** section and change the branch from `main` to `gh-pages`. Save it.

[image 2]

> A more detailed explanation about how to change your github page branch can be found in github docs:
>
> [Configuring a publishing source for your GitHub Pages site](https://docs.github.com/en/free-pro-team@latest/github/working-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)

### Optional: Define main as a protected branch

Let's avoid unexpected commits on `main` branch. So:

1. Go to your repository and then Settings. Url is `https://github.com/<USERNAME>/<USERNAME>.github.io/settings`
2. Choose Branches option
3. Add rule
    1. For "Branch name pattern", put `main`
    2. Check "Require pull request reviews before merging"
    3. Check "Require status checks to pass before merging"
        1. Check "Require branches to be up to date before merging"
        2. Check "  Page Build"
        3. Check "github/pages"
    4. "Create" and then "Save changes".

> Again, a more detailled explanation is available on github docs:
>
> [Configuring protected branches](https://docs.github.com/en/free-pro-team@latest/github/administering-a-repository/configuring-protected-branches)

## Travis configuration

### Set up your travis-ci account

> 1. Go to Travis-ci.com and Sign up with GitHub.
> 2. Accept the Authorization of Travis CI. You’ll be redirected to GitHub. For any doubts on the Travis CI GitHub Authorized OAuth App access rights message, please read more details below
> 3. Click on your profile picture in the top right of your Travis Dashboard, click Settings and then the green Activate button, and select the repositories you want to use with Travis CI.
>
> Extracted from [Travis CI Tutorial](https://docs.travis-ci.com/user/tutorial/#to-get-started-with-travis-ci-using-github)

In our case, we want to be sure our `<username>.github.io is activated.

### Github token

I believe one of the keys of my success is the steps described by CreatureSurvive in his/her very concise and useful post [Hugo on GitHub Pages using Travis-CI for deployment](https://creaturesurvive.github.io/repo/blog/hugo-on-github-pages-using-travis-ci-for-deployment). All needed information is there, but I will rephrase and detail some steps below.

Github tokens are a way to give permissions to other agents to access/act in your repositories without giving them your password.

#### Create a new github token

1. To generate a new token, go to [your Settings (click in your avatar on the top right and then Settings) -> "Developer settings" option -> "Personal access tokens" option -> "Generate new token" button](https://github.com/settings/tokens/new).

2. Choose a name (I recommend something meaningful like `travis-ci-token`) and select repo checkbox (travis will commit changes for you).

[image 3]

3. Generate the token (button in the bottom) and store the value.

### Set the github token in travis

1. Go to `https://travis-ci.com/github/<USERNAME>/<USERNAME>.github.io` and in "more options" choose Settings.
2. Add a new environment variable called `GITHUB_TOKEN` and insert the token value.
    1. Ensure Display value in build log is unchecked.
    2. Optionally: you can restrict the access to branch `main`.
    3. Save it.

[image 4]

## Hugo configuration

Hugo offers an excellent [quick start guide](https://gohugo.io/getting-started/quick-start/), but for the purpose to keep everything in one tutorial, I will detail the steps here.

### Install Hugo

On macOS, I recommend using [Homebrew](https://brew.sh/). For windows or linux, please take a look on [Install Hugo doc](https://gohugo.io/getting-started/installing).
```shell
brew install hugo
```

### Create the blog

As we already set up [our repository locally](#create-the-repo-locally), we will now set hugo there

```shell
cd tomasalmeida.github.io
hugo new site --force ./ # --force is needed as the folder is not empty
```

### Define a theme

In https://themes.gohugo.io, you have several options to choose. I will use the same as in the Hugo tutorial: [Ananke Gohugo Theme](https://themes.gohugo.io/gohugo-theme-ananke/) 

```shell
git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke
git add themes 
git commit -m "Adding a submodule for the theme ananke" 
```

### Define a basic configuration for the site

Let's add the basic configuration for the blog, as the url, the title and the theme we downloaded in the previous step.

For that open the `config.toml` and edit to to something similar to:

```text
baseURL = "https://tomasalmeida.github.io/"
languageCode = "en-us"
title = "Tomás Dias Almeida notes"
theme = "ananke"
```

Commit the changes:
```shell
git add config.toml
git commit -m "Adding blog basic configuration"
```

### Create your first post

Well, I am creating this post right now. As you can see I like to have the date in the name for organization. I believe it goes against SEO, but it is easier to maintain control over the posts.

```shell
hugo new posts/2020/12/creating-blog-using-hugo-travis-github.md
```

As you can see, a new file is created with something like this:
```text
---
title: "Creating Blog Using Hugo Travis Github"
date: 2020-12-24T12:58:00+01:00
draft: true
---
```

Brief explanation:
* **title**: it is an auto generated title for your post.
* **date**: when the file was created
* **draft**: true means do not publish it.

If you want to see the changes directly in [http://localhost:1313](http://localhost:1313/) if you can run

```shell
hugo server -D
```

Once your finish your post, it is time to commit everything (do not forget to change the draft option to false)

Commit the changes:
```shell
git add .
git commit -m "My first blog post about creating a blog and its first blog post"
```

## Create Travis configuration

Last and easiest part, we will connect all together. Under the root of your project create the `.travis.yml` file with this content:

```yaml
---
install:
  - curl -LO https://github.com/gohugoio/hugo/releases/download/v0.79.1/hugo_0.79.1_Linux-64bit.deb
  - sudo dpkg -i hugo_0.79.1_Linux-64bit.deb

script:
  - hugo

deploy:
  provider: pages
  skip_cleanup: true
  local_dir: public
  github_token: $GITHUB_TOKEN # already Set in travis-ci.org dashboard
  on:
    branch: main
```

Thanks again for CreatureSurvive and the useful post [Hugo on GitHub Pages using Travis-CI for deployment](https://creaturesurvive.github.io/repo/blog/hugo-on-github-pages-using-travis-ci-for-deployment).

Commit travis configuration file:

```shell
git add .travis.yml
git commit -m "Travis ci configuration"
```