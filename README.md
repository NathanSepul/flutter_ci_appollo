<h1>Appollo with CI/CD GitHub Actions</h1>

This is an example project, with the objective the show how to use the Github Action with [Appollo](https://github.com/Appollo-CLI/Appollo "The easy way to setup, build & release flutter apps for iOS on Linux, Windows and MacOS").  
![workflow](/.images/workflow.jpg "workflow")

<h2>Prerequisites</h2>

To Follow this tutorial you will need :
- [X] [Appollo](https://github.com/Appollo-CLI/Appollo) configured on the futur runner machine
- [x] A project [Flutter](https://docs.flutter.dev/get-started/install) directly in root repository

<h2>Configuration</h2>

To use Github Acitons there are 2 possibilities, use the GitHub's runner (paid solution) or use the self-hosted runners (free solution).
In this tutorial we will use the sel-hosted runner.

<h3>Self-hosted runner</h3>

To add a self-hosted runner to your repository you should go to settings's repository **>** Actions **>**  Runners  **>** New self-hosted runner button and follow the tutorial for your os. 

<h3>Creation of actions file</h3>

To work properly you need to create these folder at the root of project 

```
mkdir -p .github/worklows/
```
<br>
Inside workflows you will create github_actions.yml file and insert your actions.

<br>  
Exemple of workflow with Appollo

```YAML 
name : appollo ci

on: ['push']

jobs:
  check_validity_flutter:
    runs-on: [<personal_runner_label>]
    name: "Test phase" 
    steps:
      # extract repo
      - name: Checkout
        uses: actions/checkout@v3

      - name: Run unite test
        run: echo 'test flutter'
        run: flutter test


  build_ipa:
    needs: check_validity_flutter
    runs-on: [<personal_runner_label>]
    name: "Build IPA file" 
    steps:
      - name: Connection
        run : echo "Connection step"
        run : appollo signin --email <email> --password <password>

      - name: Building the IPA
        run: echo 'build ipa'
        run: appollo build start --build-type=ad-hoc <application_key>


      ##
      ## Affichage de l'ipa
      ##


      - name: Disconnection
        run : echo "Disconnection step"
        run : appollo signout


  deploy:
    needs: build_ipa
    runs-on: [<personal_runner_label>]
    name: "Publication app" 
    
    # to use only if push on 'production' branch
    if: github.ref == 'refs/heads/production'
    steps:
      - name: Connection
        run : echo "Connection step"
        run : appollo signin --email <email> --password <password>

      - name: Publication
        run : echo "on publie"
        run: appollo build start --build-type=publication <application_key>
      
      - name: Disconnection
        run : echo "Disconnection step"
        run : appollo signout
```

In this exemple we have 4 parametres:
- <*personal_runner_label*> is the self-hosted runner label defined when it was created
- <*email*> is the email to connect to your account on appollo
- <*password*> is the password to connect to your account on appollo
- <*application_key*> is the key off your application. 

<br>
If you forgot the application's key you can use this following command : 
```
appollo app ls
```

<h2>Usage</h2>

Now that all is configured you doens't need to do anything else.  
The previous worflow is call on each push no matter the branch because we specify *on: ['push']*.  
However the last job is cald only if there are a push on *production* branch

<h3>View the actions</h3>

When you push your code on Github you can show the workflow executed or in execution in the section *Actions* of the repository
![Go to action](/.images/actions_bar.jpg "Go to action")

If the unit tests has been successfully passed and the build ipa successed too you got te url to get the IPA, either to download it, or to install it if opened from an iOS device.

Finally if the push was on *production* branch the workflow will publish your app on App Store.

And that's it with this tutorial you learn how to use Appollo with Github Actions.