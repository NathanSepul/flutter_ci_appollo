=================================
Appollo with CI/CD GitHub Actions
=================================
This is an example project, with the objective to show how to use Github Actions with `Appollo <https://github.com/Appollo-CLI/Appollo>`_.
for releasing iOS apps with a CI solution.  

.. image:: /.images/workflow.jpg
    :align: center
    :width: 100%

-------------
Prerequisites
-------------

To Follow this tutorial you will need :

* An appollo account linked to your Apple Developer Account. `Learn how to setup Appollo <https://appollo.readthedocs.io/en/master/tutorial/2_configure_app_store_connect.html>`_.
* A `Flutter <https://docs.flutter.dev/get-started/install>`_ project located at the root of your git repository.

-------------
Configuration
-------------

To use Github Actions there are 2 possibilities, use the GitHub's runner (paid solution) or use the self-hosted runners (free solution).
If you want use the free solution, you can add a self-hosted runner to your repository by going to repository's settings  **>** Actions **>**  Runners  **>** New self-hosted runner button and follow the tutorial for your os. 


.. note:: 
  When using a GitHub runner or self-hosted runner, The only difference is the way to call the runner.


<h3>Creation of actions file</h3>

To work properly you need to create this folder at the root of your project 

.. code-block::

    mkdir -p .github/worklows/


Inside workflows you will create a github_actions.yml file. This is where we will add the actions.
Here is an example :

.. code-block:: yml
  name : appollo ci

  on: ['push']

  jobs:
    check_validity_flutter:
      runs-on: [<personal_runner_label>]
      name: "Run tests" 
      steps:
        # extract repo
        - name: Checkout
          uses: actions/checkout@v3

        - name: Run unit test
          run: flutter test


    build_ipa:
      needs: check_validity_flutter
      runs-on: [<personal_runner_label>]
      name: "Build IPA file"
      if: github.ref != 'refs/heads/production'
      steps:
        - name: Install Appollo
          run: pip3 install -y Appollo

        - name: Connection
          run : appollo signin --email <email> --password <password>

        - name: Building the IPA
          run: appollo build start --build-type=ad-hoc <application_key>

        - name: Disconnection
          run : appollo signout


    deploy:
      needs: build_ipa
      runs-on: [<personal_runner_label>]
      name: "Publication app" 
      
      # only do this if we pushed on 'production' branch
      if: github.ref == 'refs/heads/production'
      steps:
        - name: Install Appollo
          run: pip3 install -y Appollo
          
        - name: Connection
          run : appollo signin --email <email> --password <password>

        - name: Publication
          run: appollo build start --build-type=publication <application_key>
        
        - name: Disconnection
          run : appollo signout

In this exemple we have 4 parameters:

* <*personal_runner_label*> is the self-hosted runner label defined when it was created
* <*email*> is the email to connect to your account on appollo
* <*password*> is the password to connect to your account on appollo
* <*application_key*> is the key off your application. 

.. note:: 
  If you forgot the application's Appollo key you can use this following command :  ``appollo app ls``

-----
Usage
-----

Now that all is configured you don't need to do anything else. The previously made worflow is called on each push no matter the branch because we specified ``on: ['push']`` in our configuration file.  
However the last jobs are only called if there was a push on the ``production`` branch and the second job isn't called in this case.

^^^^^^^^^^^^^^^^
View the actions
^^^^^^^^^^^^^^^^

When you push your code on Github you can show the workflow executed or in execution in the section *Actions* of the repository
.. image:: /.images/actions_bar.jpg
    :align: center

If the unit tests have been successfully passed and the build ipa succeeded you get back the url to the IPA, either to download it, or to install it if opened (in safari) from an iOS device.

Finally if the push was on the ``production`` branch the workflow will publish your app on the App Store directly. You can then either test your application through testflight or submit the latest version to Apple.

And that's it with this tutorial you have learned how to use Appollo with Github Actions.

-------------
Documentation
-------------
We propose 3 others examples of solution with other CI tools:

* [GitLab Ci](https://gitlab.com/NathanSepul/flutter_ci_appollo)
* [Bitbucket Pipelines](https://bitbucket.org/appollo-ci-cd/flutter_appollo_ci)
* [Circle Ci](https://github.com/NathanSepul/flutter_appollo_circle_ci)
