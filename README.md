<h1>Appollo with CI/CD GitHub Actions</h1>

This is an example project, with the objective the show how to use the Github Action with [Appollo](https://github.com/Appollo-CLI/Appollo "The easy way to setup, build & release flutter apps for iOS on Linux, Windows and MacOS").

![workflow](/.images/workflow.jpg "workflow")

<h2>Prerequisites</h2>

To Follow this tutorial you will need :
- [X] [Appollo](https://github.com/Appollo-CLI/Appollo) configured on the futur runner machine
- [x] A project [Flutter](https://docs.flutter.dev/get-started/install) directly in root repository
- [x] 

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

inside workflows you will create github_actions.yml file and insert your actions. 
exemple of workflow with appollo

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
    
    # used only if push on 'production' branch
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
- &lt*personal_runner_label*&gt is the self-hosted runner label defined when it was created
- &lt*email*&gt is the email to connect to your account on appollo
- &lt*password*&gt is the password to connect to your account on appollo
- &lt*application_key*&gt is the key off your application. 


If you forgot the application's key you can use this following command : 
```
appollo app ls
```

<h2>Usage</h2>