Tests 
===================================
.. tests:

To ensure the application works after each update to the project, we wrote tests that will be running after each push on the Github Repository. The workflows for all of the tests can be found inside the .github\workflow folder. All of the PHPUnit tests can found inside the tests folder split between Unit and Feature tests.
Also, pulling up a random commit should show the tests being run and (hopefully) succesfully being completed.

.. _workflow:
Workflow PHPUnit Tests
--------
Workflow for GitHub Actions
This workflow is triggered when a push event occurs on the repository. It consists of a single job that runs on an Ubuntu machine. The steps in the job are as follows:

1.	Retrieve the code from the repository using the actions/checkout@v2 action.
2.	Copy a file named .env if it does not exist, otherwise copy the file from .env.example.
3.	Install dependencies using Composer.
4.	Generate a unique key for the application using the command php artisan key:generate.
5.	Change the permissions of the storage and bootstrap/cache folders so that they have read and write access for everyone.
6.	Create a folder named database and within it, create an empty SQLite file database.sqlite.
7.	Compile assets using npm.
8.	Run tests using PHPUnit, using a SQLite database and setting other configuration settings.


.. code-block:: yaml
:linenos:
name: PHP tests
on: [push]
jobs:
  unit:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Copy .env
      run: php -r "file_exists('.env') || copy('.env.example', '.env');"
    - name: Install Dependencies
      run: composer install -q --no-ansi --no-interaction --no-scripts --no-progress
    - name: Generate key
      run: php artisan key:generate
    - name: Directory Permissions
      run: chmod -R 777 storage bootstrap/cache
    - name: Create Database
      run: |
        mkdir -p database
        touch database/database.sqlite
    - name: Compile assets
      run: |
        npm install
        npm run production
    - name: Execute tests (Unit and Feature tests) via PHPUnit
      env:
        DB_CONNECTION: sqlite
        DB_DATABASE: database/database.sqlite
        CACHE_DRIVER: array
        SESSION_DRIVER: array
        QUEUE_DRIVER: sync
      run: vendor/bin/phpunit



