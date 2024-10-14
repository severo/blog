---
sidebar: false
---

# Create a static Hugging Face space with React

In this tutorial, let's see how to:
- create a React app using Vite (with support for TypeScript)
- host it as a GitHub repository
- deploy it as a static Hugging Face space

The code is  and the space is https://huggingface.co/spaces/severo/hello-react-app

## Create the React app

In a local directory, run:

```bash
cd /tmp
npm create vite@latest hello-react-app -- --template react-ts
```

<div class="note">

You can replace `react-ts` with any other template provided by Vite: `vanilla`, `vanilla-ts`, `vue`, `vue-ts`, `react`, `react-ts`, `react-swc`, `react-swc-ts`, `preact`, `preact-ts`, `lit`, `lit-ts`, `svelte`, `svelte-ts`, `solid`, `solid-ts`, `qwik`, `qwik-ts`.

</div>

Ensure it works:

```bash
cd /tmp/hello-react-app
npm install
npm run dev
```

## Create a GitHub repository

At https://github.com/new, create a new repository. Mine is https://github.com/severo/hello-react-app.

Set it as the remote of your local repository:

```bash
cd /tmp/hello-react-app
git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:severo/hello-react-app.git # <= adapt
git push -u origin main
```

## Create a static Hugging Face space

Create a new space on Hugging Face at https://huggingface.co/new-space. Select "Static" / "Blank" as the Space SDK. Mine is https://huggingface.co/spaces/severo/hello-react-app.

## Create a Hugging Face access token

Create an access token at https://huggingface.co/settings/tokens, to be able to write to the space files. Click "Create new token", select "Fine-grained", set a name, eg "Write to hello-react-app", in "Repository permissions" search for you space, eg "severo/hello-react-app", and enable the checkbox "Write access to contents/settings of selected repos". Finally, click "Create token". Copy the value and keep it safe for the next step.

## Create a GitHub Action

Create a new file at `.github/workflows/huggingface.yml` with the following content:

```yaml
name: Deploy to Hugging Face
on:
  push:
    branches:
      - main
  workflow_dispatch:

env:
  STATIC_SPACE: "severo/hello-react-app"
  DIRECTORY: "dist"

jobs:
  deploy:
    # inspired by https://huggingface.co/blog/severo/build-static-html-spaces
    runs-on: ubuntu-latest
    steps:
      - name: Set up Python 3.12
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - name: Install huggingface_hub
        run: pip install --upgrade "huggingface_hub[cli]"
      - name: Checkout the code
        uses: actions/checkout@v4
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
      - name: build the app
        run: npm install && npm run build
      - name: Copy the README.md from the space to the public folder
        run: huggingface-cli download --token=${{ secrets.HF_TOKEN }} --repo-type=space --local-dir=${{env.DIRECTORY}} ${{env.STATIC_SPACE}} README.md
      - name: Delete hfh cache
        run: rm -rf ${{env.DIRECTORY}}/.cache
      - name: Push to HF space (deleting the previous files)
        run: huggingface-cli upload --token=${{ secrets.HF_TOKEN }} --repo-type=space ${{env.STATIC_SPACE}} ${{env.DIRECTORY}} . --delete "*"
```

Set your Hugging Face space to `STATIC_SPACE`.

Also, in the GitHub repository, go to "Settings" > "Secrets and variables" > "Actions" > "New repository secret" and add a secret called "HF_TOKEN" and paste the token you created in the previous step.

Commit the new file and push it to GitHub:

```bash
git add .github/workflows/huggingface.yml
git commit -m "Add GitHub Action to deploy to Hugging Face"
git push
```

The action is triggered on GitHub, and it will build and deploy the app to the Hugging Face space.

## Conclusion

That's it! You have a React app hosted on GitHub and deployed as a static Hugging Face space. You can see it at https://huggingface.co/spaces/severo/hello-react-app.
