### initialize the directory with a package.json file

```bash
npm init -y
```

### create an action metadata file

```bash
name: 'Hello World'
description: 'Greet someone and record the time'
inputs: # input parameters are stored as environment variables
  who-to-greet:  # id of input
    description: 'Who to greet'
    required: true
    default: 'World'
outputs:
  time: # id of output
    description: 'The time we greeted you'
runs:
  using: 'node12'
  main: 'index.js'
```

### add the actions toolkit

```bash
# the @actions/core package provides an interface to the commands in the workflow and helps with input and output variables
npm install @actions/core

# the @actions/github package returns the Octokit REST client and GitHub actions contexts
npm install @actions/github
```

[actions/toolkit repository](https://github.com/actions/toolkit)

### writing the js code

```bash
# the code is in the index.js file
# prints hello $person in a debug message in the log
# the script gets the current time and sets it as an output variable for our action to use
# GitHub Actions provide context information about the webhook event, Git refs, workflow, action, and the person who triggered the workflow
const core = require('@actions/core');
const github = require('@actions/github');

try {
  // `who-to-greet` input defined in action metadata file
  const nameToGreet = core.getInput('who-to-greet');
  console.log(`Hello ${nameToGreet}!`);
  const time = (new Date()).toTimeString();
  core.setOutput("time", time);
  // Get the JSON webhook payload for the event that triggered the workflow
  const payload = JSON.stringify(github.context.payload, undefined, 2)
  console.log(`The event payload: ${payload}`);
} catch (error) {
  core.setFailed(error.message);
}
```

### commit and push to your own github repo

```bash
git add .
git commit -m "my first action"
git tag -a -m "My first action release" v1 # its best practice to add a version tag for releases of your action, so others can use it too!
git push --follow-tags # cancel and enter github creds
```

### Create workflow (private)

```yaml
on: [push]

jobs:
  hello_world_job:
    runs-on: ubuntu-latest
    name: A job to say hello
    steps:
      # To use this repository's private action,
      # you must check out the repository
      - name: Checkout
        uses: actions/checkout@v2 # This action checks-out your repository under $GITHUB_WORKSPACE, so your workflow can access it.
      - name: Hello world action step
        uses: ./ # Uses an action in the root directory
        id: hello
        with:
          who-to-greet: 'Chad Crowell' # change it to your name
      # Use the output from the `hello` step
      - name: Get the output time
        run: echo "The time was ${{ steps.hello.outputs.time }}"
```

### Create workflow (public)

When an action is in a private repository, the action can only be used in workflows in the same repository. Public actions can be used by workflows in any repository.

```bash
on: [push]

jobs:
  hello_world_job:
    runs-on: ubuntu-latest
    name: A job to say hello
    steps:
    - name: Hello world action step
      id: hello
      uses: actions/javascript-actions@v1
      with:
        who-to-greet: 'Chad Crowell'
    # Use the output from the `hello` step
    - name: Get the output time
      run: echo "The time was ${{ steps.hello.outputs.time }}"
```

### Commit and push again

```bash
git add .
git commit -m "added workflow"
git push origin master
```

