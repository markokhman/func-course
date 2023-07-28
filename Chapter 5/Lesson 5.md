## Chapter 5. Lesson 5. Deploying Dapp to Github
###### tags: `Chapter 5`


GitHub Pages is a free service for open source projects that allow them to publish static websites directly from GitHub repository.
Facts about GitHub pages:
1. GitHub Pages supports static websites only, that only run client-side, and there is no server-side rendering. 
2. GitHub Pages gives many community governance features for free. 
3. Anyone from the community is unhappy how dApp is governed, can always fork the GitHub repository and create own independent client. 

### Deploying the Web Application to GitHub Pages

First, we need to create a new repository and push our code to it. 

Repository name will be **first_contract_front_end** and repository will be **public**.

We should type some commands in root of our repository:

``` 
git init
git add .
git commit -m "Initial commit"
git remote add origin "Link of your repository here in format https://github.com/user_name/repository_name.git"
git push origin master
```

How to get from user_name and repository_name link to our future site?
The link to the site will be like this: user_name.github.io/first_contract_front_end/

1. Create **tonconnect-manifest.json** in folder **public** and put this code here
``` 
{
    "url": "https://join.toncompany.org",
    "name": "TON&Co. Tutorial",
    "iconUrl": "https://user_name.github.io/first_contract_front_end/icon.png"
}
``` 
2. Save file **icon.png** in folder **public**.
3. Put in **src/main.tsx**
``` 
const manifestUrl = "https://nick_name.github.io/first_contract_front_end/tonconnect-manifest.json";
``` 
4. In **src/vite.config.ts** put
```
base: "/first_contract_front_end",
```

5. Now we're ready to run
```
yarn build
```

### Github workflow

We don't want to do this command every time. That's why we are going to create workflow that is going to do this job every time we push to repository.

1. Create a repository **.github** in root repository.
2. Create a repository **workflows** in **.github** repository.
3. Create **deploy.yml** file inside **workflows**.
4. Paste in file **deploy.yml** following:
```
name: Deploy

on:
  push:
    branches:
      - master


jobs:
  build:
    name: Build
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repo
        uses: actions/checkout@v2

      - name: Setup Node
        uses: actions/setup-node@v1
        with:
          node-version: 16

      - name: Install dependencies
        uses: bahmutov/npm-install@v1

      - name: Build project
        run: npm run build

      - name: Upload production-ready build files
        uses: actions/upload-artifact@v2
        with:
          name: production-files
          path: ./dist

  deploy:
    name: Deploy
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/master'

    steps:
      - name: Download artifact
        uses: actions/download-artifact@v2
        with:
          name: production-files
          path: ./dist

      - name:
        uses: peaceiris/actions-gh-pages@v3
        with: 
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
          
```
5. Run
```
git add .
git commit -m "Workflow"
```

Then get back to Repository --> Settings --> Actions --> General --> Workflow permissions 
and pick **read and write permissions**.

6. Run
```
git push origin master
```

After that we can get back to Repository --> Actions and track our workflow resulted in creating new branch **gh-pages**.
Let's go to Settings --> Pages and choose what branch should be used to publish GitHub Pages site. We pick **gh-pages**.
Now we're heading back to Actions and we see that one more action is running. This action is going to deploy content of our folder in our subdomain.
We can check the result: Settings --> Pages --> Visit site. And we see it's working just like on our local machine. Contratulations, we deployed our first application to production.
