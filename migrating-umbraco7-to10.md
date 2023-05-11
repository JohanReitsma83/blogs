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

# V7 Steps!

## 1.Get everything locally working on your machine!
So first we want everything locally working on your machine. I hope you have the project in a GIT repository bevcause it will give you an overview of viewing your changes and see if you are forgetting something in the future.

So you have the source checked out? Nice. Now get a backup of the production database.

### why production database? Why not development, test or acceptance database?
In the migrations of a lot of projects I have learned that migrating Test or any other Database is missing user generated content that can be a problem if you don't find it out early in your migration steps!.

So you've got a backup of production? Now use SQL Developer edition or a test database server to host that database. 

## 2.Ajust the connection string
Make sure the connection string is changed, and keep this in a notepad our something, we are gonna need it multiple times!
<add name="umbracoDbDSN" connectionString="server=servername;database=migration-customer;user id=username;password='******'" providerName="System.Data.SqlClient" />

## 3.Download the media folder
Download the media folder from production back into your local media folder. Why? This will help to test if everything is working locally before the upgrade! And you need this in your V10 to compare it with your production running website

## 4.Download the forms json files from App_Data
We also need to download the forms files, this is needed because the Json files needs to be migrated into the database. That is done later on in the Umbraco V8 version and can't be done in V9 or up versions!

## 5.Run the site locally
Don't fotget to change the domain name on your root node to the localhost version, then you should see a complete working version of your website locally!.

# V8 Step!
so now i'm running umbraco V8 in an IIS instance locally (love that way) but you can do this also on a test server. I'm going to go with local iis in this blog.

## 1. Create a site in IIS
Bind your website to the V8 and add a binding like upgrade.localhost. 

## 2. changes to the web.config
Change the connection string in the web.config to the working V7 version. Note, you also need to fill in 
<add key="Umbraco.Core.ConfigurationStatus" value="7.15.7" /> to the old v7 version! Keep in mind, this needs to be the correct version of V7.

##.
