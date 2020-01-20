# Python Environment Project

## Creating Development environment
The first step I took to provision the development environment for the Python application was to create the machine using Vagrant and provision it using a bash shell script.
The commands I needed to run to allow the application to work on the Ubuntu machine were:
`sudo apt update -y`
`sudo apt install python3-pip -y`
`pip3 install -r /home/vagrant/app/requirements.txt`
`mkdir /home/vagrant/Downloads`

These commands will update the package list, install the python pip module, pip install all of the required python libraries and then make a directory which is needed for the app to generate the CSV file when it is run successfully.

## Provisioning with Chef
The next step was to use the provision shell script I previously created and convert it into a cookbook which I could then use to provision my Ubuntu machine instead.
To do this I created a *cookbooks* folder and from here I ran:
`chef generate cookbook It_Jobs`
This created all the default files I needed for my cookbook. The main files I had to edit to do the provisioning were the *recipe/default.rb* to download the required packages and complete the machine provisioning, *spec/unit/recipes/default_spec.rb* for the Chef unit tests and *test/integration/default/default_test.rb* for the Chef integration test.
To test the cookbook was working I ran the unit test: `chef exec rspec spec` and the integration test `kitchen verify`.
In the integration test there was an issue I had where the *requirements.txt* file was not being found this was because the kitchen test was not syncing the files as it does with Vagrant.
After I edited the *recipe/default.rb* file to get all the packages and make the required directories I had to edit my *Vagrantfile* to use this cookbook to complete the provisioning. I tested this stage worked by running:
`vagrant up`
`vagrant ssh`
From there inside the machine I navigated to the Python Applications folder and ran the app to make sure it still worked

## Create AMI using Packer
The next step was to create a Packer file which can create the AMI (Amazon Machine Images) using the cookbook I created. Packer is a JSON file which is where we specify the options of the Amazon Machine we are trying to spin up. In the *packer.json* file I edited the *provisioners* section to use the cookbook I had created and also I had to add options to sync the Python application to the Ubuntu machine as it did not work using the commands I had in my cookbook. To create the AMI I had to run:
`packer build packer.json`
When I ran this command I got an error which I did not manage to fix:
`Build 'amazon-ebs' errored: No valid credential sources found for AWS Builder`
I think that this was an issue I had with my AWS access keys.

## Building Python cookbook Pipelines
The final step for this project was to create the pipelines which will test the code for the development environment then push it from a *developer* branch in **git** and push it the *master*. Then I should be able to create a Jenkins job which tests the Python application and uses the Packer file to create the AMI if the tests were successful.
To start I created a Github repository where I pushed all the code for my cookbook, https://github.com/hfiaz/python_cookbook_repo. Then I had to create the Jenkins job which tests the cookbook in the *developer* branch if this test was successful it triggers the second Jenkins job I created which will take all the code from the *developer* branch and merge it to *master* branch in my Github repository.
The final Jenkins job I needed to create is the one that would use packer to build the AMI once both previous Jenkins job have passed their test successfully. One main thing I had to remember to do was insert the Github webhooks into my repository to allow Jenkins to listen for changes being made to the repository.
For this part of the project I was not able to fully test everything was working as I was having an issue with Jenkins being able to use Chef for me to test my cookbook to run the pipeline. Because of this I also did not manage to create the Jenkins job which would use packer and create the AMI.

![Pipeline Diagram](/images/pipeline_diagram.JPG)
