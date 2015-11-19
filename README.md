# Up and running

Bundle project to install dependencies. To create a new post:

`
bundle exec octopress new post "My New Post"
`

Modify the new post file in _post directory.

To view site locally:

`
bundle exec jekyll serve
`

Then point browser to http://localhost:4000

To deploy:

* Commit changes and push to master
* Deploy site:

`
bundle exec octopress deploy
`

*Note*: This command reads _deploy.yml configuration to deploy to condevhi.github.io repository.
