---
layout: post
title: "Better Automated Testing"
date: 2017-02-27
modified: 2017-02-27
meta: Using bash scripts for better automated testing
summary: Where I discuss FED automated tests and a consistent method to implement them
---

I'm a stickler for good automated testing and being consistent in writing good code. When I work in other companies environments I always like to try and imbue in the collegues around me good testing practices and habits.

Part of this rhetoric is to define project specific automated tests that are easily repeatable and configurable. There are many tests you *may* want to do in a project, my tests stub usually starts with the following.

* Mocha (or whatever your preferred test coding framwork is)
* Istanbull (code coverage tests)
* Javascript lint checks (eslint is my preferred option)
* SASS lint
* HTML validation tests with Valimate
* Broken Links Checker
* Accessibility Checks with Pa11y
* Vision Checks (using my custom process)


- write bash script for full tests
- write small script for quick tests run on git hook.
- bash script that gets run periodically and notifies you if you haven't run your tests recently


``` bash
#!/bin/bash
# Runs ALL TESTS

# Define list of urls for tests
urls=(index register no-password access-granted create-account check-email start take-a-photo photo-audit nino update-address complete cookies cookies-table 404 500)
disabilities=(achromatomaly achromatopsia deuteranomaly deuteranopia protanomaly protanopia tritanomaly tritanopia )

echo "Please Note: An internet connection is REQUIRED for all tests to run successfully"

cd ../

# Mocha unit tests
echo "Starting Mocha tests"
npm run test

# Start the test server in background
json-server --port 3004 testing/db.json &

# Start Node in test mode in background
npm run testing --scripts-prepend-node-path &

# Wait for the Node Server to start up
echo "Waiting for Node to start..."
sleep 5

# Javascript Lint checks
echo "Starting esLint..."
eslint */*.js

# Sass Lint checks
echo "Starting Sass Lint..."
sass-lint -c testing/configs/.sass-lint.yml 'assets/scss/*.scss' -v -q
# open -g testing/linting/sass-lint.html

# Run Valimate HTML validation tests
cd testing/configs
valimate

cd ../

# Run Broken Link Checker
for i in "${urls[@]}"
do
    blc http://localhost:3000/$i
done

# Pa11y accesibility checks
echo "Starting Pa11y..."
for i in "${urls[@]}"
do
    pa11y http://localhost:3000/$i --reporter html > accessibility/$i.html
    # open -g accessibility/$i.html
    echo $i processed
done
echo "Finished Pa11y"

#Delete existing shots
rm -rf -- shots
#Re-create the shots folder
mkdir -p shots
# change directory in to shots folder and take screenshots at multiple resolutions.
cd shots

for i in "${urls[@]}"
do
    pageres http://localhost:3000/$i 320x480 360x640 768x1024 1024x768 1280x800
done

#Starting Vision checks...
# echo "Starting Vision Checks..."
# for i in "${urls[@]}"
# do
#     for j in "${disabilities[@]}"
#     do
#         # open -g http://localhost:3000/$i/$j
#         #screen shot app here
#         echo $i $j processed
#     done
# done

echo "Tests Complete"

# Kill the running processes afterwards
# Node_port=3000
# DB_port=3004
# lsof -i tcp:${Node_port} | awk 'NR!=1 {print $2}' | xargs kill
# lsof -i tcp:${DB_port} | awk 'NR!=1 {print $2}' | xargs kill
```