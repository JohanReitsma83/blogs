So, here's another blog for migrating Umbraco 7 to 10. This time its from my perspective and I'm not saying this is the way working for you but maybe you can use some of my solutions for migrating.

# What are we going to upgrade? 
in basics we upgrade an Umbraco 7 build website that uses modelsbuilder to a new version using the following steps:
- Make sure you are on the latest version of Umbraco 7!
- Have a local version of Umbraco 8, take the last one -> download from here: https://our.umbraco.com/download/releases/8188
use the zip, you only need this as step between the actual migration to v10
- and start a new project using Umbraco 10 Template
This is the short version, but below I will describe all the steps that I have done for multiple steps so you can maybe use this

# Umbraco 7 - Whats up with that?
So You have an Umbraco 7 website and need to upgrade because updates will stop in september 2023, you want to have the costs so low as possible otherwise the client doesn't want to pay and maybe wants to leave you. Then the crutial rule is to migrate the website AS IS.
That means we are not going to rebuild stuff, we are also not going to improve stuff and it will be an exact copy of the site in the Frontend. In the CMS there can be some differences but we will talk about this later.

So tell your client it's only an update to the latest (LTS) version of Umbraco so they can keep there site and have security updates available for them for the next couple of years.
From the new version you can upsell some new features like Multilingual support or improved editing experience. 

#How to start?

## Get everything locally working on your machine!
So first we want everything locally working on your machine. I hope you have the project in a GIT repository
