**Migrating from GitHub Enterprise Server to**  
**GitHub Enterprise Cloud**  
GitHub Enterprise Server is the on-premises Git repository hosting offering from GitHub. Large  
organizations commonly run GitHub Enterprise Server for improved control and security over  
their code repositories. With Enterprise Server, you can limit access to a private network, set  
rules for creating and accessing repositories, use SAML for single sign-on across your  
organization, and get access to premium GitHub support. You can also migrate to different  
hardware as your team and repositories grow, making GitHub Enterprise Server a powerful tool  
for teams with specific compliance and infrastructure requirements.  
GitHub also offers an Enterprise Cloud option that gives you dedicated enterprise resources but  
runs on GitHub’s hardware. With a 99.95% uptime SLA and access to GitHub’s built-in security  
features, Enterprise Cloud is an appealing offering for organizations looking to reduce the  
overhead of maintaining their own infrastructure. It allows you to restrict user access based on a  
user’s network and add SAML sign-on, while also removing the need for your team to manage  
potentially expensive infrastructure.  
For many organizations, starting with GitHub Teams or GitHub Enterprise Cloud makes sense. If  
you eventually need more control over your servers, it's relatively straightforward to migrate data  
using GitHub’s API. However, migrating from GitHub Enterprise Server to GitHub Enterprise  
Cloud can prove to be significantly more challenging, as detailed below.

**The Challenge of Migrating from GitHub Enterprise Server**  
**to GitHub Enterprise Cloud**  
A git repository tracks changes to all the files in a project. Typically, software projects have a  
.git/ directory inside their codebase that includes all git records. Because git is an open-source  
standard, there’s nothing inherently different about a git repository hosted on your server,  
GitHub’s servers, or even a static file hosting solution like Amazon S3.  
The challenge of migrating between git hosting providers is that many hosts offer additional  
features that improve your team's workflow and collaboration. For example, GitHub includes  
issues, pull requests, metadata, and projects. While both GitHub Enterprise Server and GitHub  
Enterprise Cloud include these features, GitHub does not provide an official method to migrate  
this data between the two systems.

**Attempt 1: GitHub Enterprise Server Export**  
After reviewing GitHub's Enterprise Server documentation, I first attempted to use GitHub’s  
Enterprise export command-line tool to export my Enterprise Server data. This tool is designed  
for migrating data from one GitHub Enterprise server to another, but I hoped to import the same  
data into GitHub Enterprise Cloud or a GitHub Team account.When you run the export, the tool generates a folder with JSON files and subdirectories for each  
repository you exported:

While this contains all the repository information, there’s no easy way to import this data into  
GitHub Enterprise Cloud. GitHub’s import API endpoint uses the repository’s git URL to pull  
data from an external version control system and import it into GitHub. Unfortunately, this  
method requires writing custom code to parse the Enterprise Server export data, as the format is  
undocumented, and it’s unclear if the data contains elements that can’t be replicated in GitHub  
Enterprise Cloud.  
Before diving into custom code, I decided to explore GitHub’s Importer tool.

**Attempt 2: GitHub Importer**  
GitHub provides an Importer tool that allows you to migrate repositories into GitHub Cloud by  
entering a repository URL and user credentials. This method is designed for ease, but it doesn’t  
fit well with GitHub Enterprise Server. Most Enterprise Server installations are on private  
networks, and if they are not, they require private mode to be enabled.  
To use the GitHub Importer tool, your Enterprise Server repositories must be publicly available  
on the internet with read access. This requirement goes against the purpose of GitHub Enterprise  
Server, which aims to keep your repositories private and secure, so this method was not feasible  
for my migration.

**The Solution: How to Migrate from GitHub Enterprise Server to Enterprise**  
**Cloud**  
While migrating a git repository itself is straightforward, moving additional data like issues, pull  
requests, and project metadata is more complex because this data isn't stored in git. Fortunately,  
GitHub support offers a recommended two-step approach for migrating from GitHub Enterprise  
Server to GitHub Enterprise Cloud.

**Step 1: Push Each Repository to GitHub Enterprise Cloud**  
The first step in the migration process is to move your git repositories. To migrate a repository  
from GitHub Enterprise Server to GitHub Cloud, follow these steps:  
1\. Clone the repository from your GitHub Enterprise Server:  
2\. git clone git@\<YOUR\_SERVER\_URL\>:\<YOUR\_REPOSITORY\_NAME\>.git  
3\. Create a new repository in your GitHub Enterprise Cloud account with the same  
repository name. GitHub will provide the SSH address for the new repository.  
4\. 6\. Add the new repository as a remote for your cloned repository:  
5\. git remote add new-origin git@github.com:\<NEW\_REPOSITORY\_NAME\>.git  
Push the repository, including all branches and tags, to GitHub Cloud:7\. git push new-origin \--all  
8\. Refresh the page in your GitHub Cloud account to verify that all branches are transferred.  
At this point, your repository is successfully moved to GitHub Cloud. However, none of the  
other GitHub-specific data (such as issues, pull requests, and projects) has been transferred yet.

**Step 2: Use GitHub’s API to Migrate Other Data**  
To migrate additional data like issues, pull requests, and project metadata, you’ll need to use the  
GitHub API. Unfortunately, GitHub doesn’t provide a bulk migration tool for this, so each piece  
of data must be migrated manually via the API.

**Example: Migrating Issues**  
1\. First, download a list of all the issues from your GitHub Enterprise Server repository:  
2\. curl \-X GET \-u "\<YOUR\_USERNAME\>:\<YOUR\_ACCESS\_TOKEN\>" \-H "Accept:  
application/vnd.github.v3+json"  
https://\<YOUR\_ENTERPRISE\_SERVER\_URL\>/api/v3/repos/\<ORG\_NAME\>/\<REPO\_NAME  
\>/issues \--output \~/issues.json  
This will generate a file called issues.json that contains the list of issues from your  
repository.  
3\. You can then create a new issue in your GitHub Enterprise Cloud repository using the  
following curl command:  
4\. curl \-X POST \-u "\<YOUR\_USERNAME\>:\<YOUR\_ACCESS\_TOKEN\>" \-H "Accept:  
application/vnd.github.v3+json" \-d '{"body":"Its just a test  
though.","title": "This is a new issue"}'  
https://api.github.com/repos/\<ORG\_NAME\>/\<REPO\_NAME\>/issues  
5\. Repeat this process for all issues in your issues.json file. This method is a bit tedious,  
as you’ll need to loop through all issues and recreate them manually.

**Additional Data Migration**  
You can use the same method to migrate other resources, such as pull requests, projects, and  
repository settings. You’ll need to identify the correct API endpoints for each resource and use  
the data in the exported JSON files to recreate these objects in your new GitHub Cloud  
repository.  
Keep in mind that GitHub’s API does not provide an easy way to correlate the IDs of issues,  
users, comments, and other resources between GitHub Enterprise Server and Cloud. You’ll  
either need to manually map these IDs or accept the fact that some errors will occur during the  
migration process.  
Additionally, timestamps will be reset to the date each issue or pull request is migrated, which  
could affect

# Migrating repositories from GitHub Enterprise Server to GitHub Enterprise Cloud

You can migrate repositories from GitHub Enterprise Server to GitHub Enterprise Cloud, using the GitHub CLI or API.

## API

#### In this article

About repository migrations with GitHub Enterprise Importer  
Prerequisites  
Step 1\. Create your personal access token  
Step 2: Get the ownerId for the destination organization  
Step 3: Set up blob storage  
Step 4: Set up a migration source in GitHub Enterprise Cloud  
Step 5: Generate migration archives on your GitHub Enterprise Server instance  
Step 6: Upload your migration archives  
Step 7: Start your repository migration  
Step 8: Check the status of your migration  
Step 9: Validate your migration and check the error log

### About repository migrations with GitHub Enterprise Importer

You can run your migration with either the GitHub CLI or the API.

The GitHub CLI simplifies the migration process and is recommended for most customers. Advanced customers with heavy customization needs can use the API to build their own integrations with GitHub Enterprise Importer.

If you choose to use the API, you'll need to write your own scripts or use an HTTP client like Insomnia. You can learn more about getting started with GitHub's APIs in Getting started with the REST API and Forming calls with GraphQL.

To migrate your repositories from GitHub Enterprise Server to GitHub Enterprise Cloud with the APIs, you will:

1. Create a personal access token for both the source and destination organization  
2. Fetch the ownerId of the destination organization on GitHub Enterprise Cloud  
3. Set up a migration source via GitHub's GraphQL API to identify where you're migrating from  
4. For each repository you want to migrate, repeat these steps.  
   1. Use the REST API on your GitHub Enterprise Server instance to generate migration archives for your repository  
   2. Upload your migration archives to a location where they can be accessed by GitHub  
   3. Start your migration using the GraphQL API for your migration destination, passing in your archive URLs  
   4. Check the status of your migration via the GraphQL API  
   5. Validate your migration and check the error log

To see instructions for using the GitHub CLI, use the tool switcher at the top of the page.

### Prerequisites

* We strongly recommend that you perform a trial run of your migration and complete your production migration soon after. To learn more about trial runs, see Overview of a migration between GitHub products.  
* Ensure you understand the data that will be migrated and the known support limitations of the Importer. For more information, see About migrations between GitHub products.  
* While not required, we recommend halting work during your production migration. The Importer doesn't support delta migrations, so any changes that happen during the migration will not migrate. If you choose not to halt work during your production migration, you'll need to manually migrate these changes.  
* In both the source and destination organizations, you must be either an organization owner or be granted the migrator role. For more information, see Managing access for a migration between GitHub products.  
* If you use GitHub Enterprise Server 3.8 or higher, to configure blob storage for exported archives, you need access to the Management Console.

### 

### Step 1\. Create your personal access token

1. Create and record a personal access token (classic) that will authenticate for the destination organization on GitHub Enterprise Cloud, making sure that the token meets all requirements. For more information, see Managing access for a migration between GitHub products.  
2. Create and record a personal access token that will authenticate for the source organization, making sure that this token also meets all of the same requirements.

### Step 2: Get the ownerId for the destination organization

As an organization owner in GitHub Enterprise Cloud, use the GetOrgInfo query to return the ownerId, also called the organization ID, for the organization you want to own the migrated repositories. You'll need the ownerId to identify your migration destination.

#### GetOrgInfo query

query(  
  $login: String\!  
){  
  organization (login: $login)  
  {  
    login  
    id  
    name  
    databaseId  
  }  
}  
Query variable	Description  
login	Your organization name.  
GetOrgInfo response  
{  
  "data": {  
    "organization": {  
      "login": "Octo",  
      "id": "MDEyOk9yZ2FuaXphdGlvbjU2MTA=",  
      "name": "Octo-org",  
      "databaseId": 5610  
    }  
  }  
}  
In this example, MDEyOk9yZ2FuaXphdGlvbjU2MTA= is the organization ID or ownerId, which we'll use in the next step.

### Step 3: Set up blob storage

Because many GitHub Enterprise Server instances sit behind firewalls, for GitHub Enterprise Server versions 3.8 or higher, we use blob storage as an intermediate location to store your data that GitHub can access.

You must first set up blob storage with a supported cloud provider, then configure your settings in the Management Console of your GitHub Enterprise Server instance.

Note

You only need to configure blob storage if you use GitHub Enterprise Server versions 3.8 or higher. If you use GitHub Enterprise Server versions 3.7 or lower, skip to Step 4: Set up a migration source in GitHub Enterprise Cloud.

Blob storage is required to migrate repositories with large Git source or metadata. If you use GitHub Enterprise Server versions 3.7 or lower, you will not be able to perform migrations where your Git source or metadata exports exceed 2GB. To perform these migrations, update to GitHub Enterprise Server versions 3.8 or higher.

#### Setting up blob storage with a supported cloud provider

The GitHub CLI supports the following blob storage providers:

* Amazon Web Services (AWS) S3  
* Azure Blob Storage

##### Setting up an AWS S3 storage bucket

In AWS, set up a S3 bucket. For more information, see Creating a bucket in the AWS documentation.

You will also need an AWS access key and secret key with the following permissions:

{  
    "Version": "2012-10-17",  
    "Statement": \[  
        {  
            "Sid": "VisualEditor0",  
            "Effect": "Allow",  
            "Action": \[  
                "s3:PutObject",  
                "s3:GetObject",  
                "s3:ListBucketMultipartUploads",  
                "s3:AbortMultipartUpload",  
                "s3:ListBucket",  
                "s3:DeleteObject",  
                "s3:ListMultipartUploadParts"  
            \],  
            "Resource": \[  
                "arn:aws:s3:::github-migration-bucket",  
                "arn:aws:s3:::github-migration-bucket/\*"  
            \]  
        }  
    \]  
}  
Note

GitHub Enterprise Importer does not delete your archive from AWS after your migration is finished. To reduce storage costs, we recommend configuring auto-deletion of your archive after a period of time. For more information, see Setting lifecycle configuration on a bucket in the AWS documentation.

##### Setting up an Azure Blob Storage account

In Azure, create a storage account and make a note of your connection string. For more information, see Manage storage account access keys in Microsoft Docs.

Note

GitHub Enterprise Importer does not delete your archive from Azure Blob Storage after your migration is finished. To reduce storage costs, we recommend configuring auto-deletion of your archive after a period of time. For more information, see Optimize costs by automatically managing the data lifecycle in Microsoft Docs.

#### Configuring blob storage in the Management Console of your GitHub Enterprise Server instance

After you set up an AWS S3 storage bucket or Azure Blob Storage storage account, configure blob storage in the Management Console of your GitHub Enterprise Server instance. For more information about the Management Console, see Administering your instance from the Management Console.

1. From an administrative account on GitHub Enterprise Server, in the upper-right corner of any page, click .

2. If you're not already on the "Site admin" page, in the upper-left corner, click Site admin.

3. In the " Site admin" sidebar, click Management Console.

4. Log into the Management Console.

5. In the top navigation bar, click Settings.

6. Under Migrations, click Enable GitHub Migrations.

7. Optionally, to import storage settings you configured for GitHub Actions, select Copy Storage settings from Actions. For more information see, Enabling GitHub Actions with Azure Blob storage and Enabling GitHub Actions with Amazon S3 storage.

Note

After copying your storage settings, you may still need to update the configuration of your cloud storage account to work with GitHub Enterprise Importer. In particular, you must ensure that GitHub's IP addresses are allowlisted. For more information, see Managing access for a migration between GitHub products.

8. If you do not import storage settings from GitHub Actions, select either Azure Blob Storage or Amazon S3 and fill in the required details.

Note

If you use Amazon S3, you must set the "AWS Service URL" to the standard endpoint for the AWS region where your bucket is located. For example, if your bucket is located in the eu-west-1 region, the "AWS Service URL" would be https://s3.eu-west-1.amazonaws.com. Your GitHub Enterprise Server instance's network must allow access to this host. Gateway endpoints are not supported, such as bucket.vpce-0e25b8cdd720f900e-argc85vg.s3.eu-west-1.vpce.amazonaws.com. For more information about gateway endpoints, see Gateway endpoints for Amazon S3 in the AWS documentation.

9. Click Test storage settings.

10. Click Save settings.

#### Allowing network access

If you have configured firewall rules on your storage account, ensure you have allowed access to the IP ranges for your migration destination. See Managing access for a migration between GitHub products.

### Step 4: Set up a migration source in GitHub Enterprise Cloud

You can set up a migration source using the createMigrationSource query. You'll need to supply the ownerId, or organization ID, gathered from the GetOrgInfo query.

Your migration source is your organization on GitHub Enterprise Server.

#### createMigrationSource mutation

mutation createMigrationSource($name: String\!, $url: String\!, $ownerId: ID\!) {  
  createMigrationSource(input: {name: $name, url: $url, ownerId: $ownerId, type: GITHUB\_ARCHIVE}) {  
    migrationSource {  
      id  
      name  
      url  
      type  
    }  
  }  
}

Note

Make sure to use GITHUB\_ARCHIVE for type.

| Query variable | Description |
| :---- | :---- |
| name | A name for your migration source. This name is for your own reference, so you can use any string. |
| ownerID | The organization ID of your organization on GitHub Enterprise Cloud. |
| url | The URL for your GitHub Enterprise Server instance. This URL does not need to be accessible from GitHub Enterprise Cloud. |

#### createMigrationSource response

{  
  "data": {  
    "createMigrationSource": {  
      "migrationSource": {  
        "id": "MS\_kgDaACRjZTY5NGQ1OC1mNDkyLTQ2NjgtOGE1NS00MGUxYTdlZmQwNWQ",  
        "name": "GHES Source",  
        "url": "https://my-ghes-hostname.com",  
        "type": "GITHUB\_ARCHIVE"  
      }  
    }  
  }  
}  
In this example, MS\_kgDaACRjZTY5NGQ1OC1mNDkyLTQ2NjgtOGE1NS00MGUxYTdlZmQwNWQ is the migration source ID, which we'll use in a later step.

### Step 5: Generate migration archives on your GitHub Enterprise Server instance

You will use the REST API to start two "migrations" in GitHub Enterprise Server: one to generate an archive of your repository's Git source, and one to generate an archive of your repository's metadata (such as issues and pull requests).

To generate the Git source archive, make a POST request to https://HOSTNAME/api/v3/orgs/ORGANIZATION/migrations, replacing HOSTNAME with the hostname of your GitHub Enterprise Server instance, and ORGANIZATION with your organization's login.

In the body, specify the single repository you want to migrate. Your request will look something like this:

POST /api/v3/orgs/acme-corp/migrations HTTP/1.1  
Accept: application/vnd.github+json  
Authorization: Bearer \<TOKEN\>  
Content-Type: application/json  
Host: github.acmecorp.net

{  
  "repositories": \["repository\_to\_migrate"\],  
  "exclude\_metadata": true  
}  
To generate your metadata archive, send a similar request to the same URL with the following body:

{  
  "repositories": \["repository\_to\_migrate"\],  
  "exclude\_git\_data": true,  
  "exclude\_releases": false,  
  "exclude\_owner\_projects": true  
}  
Each of these two API calls will return a JSON response including the ID of the migration you have started.

HTTP/1.1 201 Created

{  
  "id": 123,  
  // ...  
}  
For more information, see Start an organization migration.

Generating the archives can take a while, depending on the amount of data. You can regularly check the status of the two migrations with the "Get an organization migration status" API until the state of the migration changes to exported.

GET /api/v3/orgs/acme-corp/migrations/123 HTTP/1.1  
Accept: application/vnd.github+json  
Authorization: Bearer \<TOKEN\>  
Host: github.acmecorp.net

HTTP/1.1 200 OK  
Content-Type: application/json

{  
  "id": 123,  
  "state": "exported",  
  // ...  
}  
For more information, see Get an organization migration status.

Note

If your migration moves to the failed state rather than the exported state, try starting the migration again. If the migration fails repeatedly, we recommend generating the archives using ghe-migrator instead of the API.

Follow the steps in Exporting migration data from your enterprise, adding only one repository to the migration. At the end of the process, you will have a single migration archive with your Git source and metadata, and you can move to step 6 in this article.

After the state of a migration moves to exported, you can fetch the migration's URL using the "Download an organization migration archive" API.

GET /api/v3/orgs/acme-corp/migrations/123/archive HTTP/1.1  
Accept: application/vnd.github+json  
Authorization: Bearer \<TOKEN\>  
Host: github.acmecorp.net

HTTP/1.1 302 Found  
Location: https://media.github.acmecorp.net/migrations/123/archive/cca2ebe9-7403-4ffa-9b6a-4c9e16c94410?token=AAAAABEWE7JP4H2HACKEGMTDOYRC6  
The API will return a 302 Found response with a Location header redirecting to the URL where the downloadable archive is located. Download the two files: one for the Git source, and one for the metadata.

For more information, see Download an organization migration archive.

After both migrations have completed and you have downloaded the archives, you can move to the next step.

### Step 6: Upload your migration archives

To import your data into GitHub Enterprise Cloud, you must pass both archives for each repository (Git source and metadata) from your machine to GitHub, using our GraphQL API.

If you're using GitHub Enterprise Server 3.7 or lower, you must first generate URLs for the archives that are accessible by GitHub. Most customers choose to upload the archives to a cloud provider's blob storage service, such as Amazon S3 or Azure Blob Storage, then generate a short-lived URL for each.

If you're using GitHub Enterprise Server 3.8 or higher, your instance uploads the archives and generates the URLs for you. The Location header in the previous step will return the short-lived URL.

You may need to allowlist GitHub's IP ranges. For more information, see Managing access for a migration between GitHub products.

### Step 7: Start your repository migration

When you start a migration, a single repository and its accompanying data migrates into a brand new GitHub repository that you identify.

If you want to move multiple repositories at once from the same source organization, you can queue multiple migrations. You can run up to 5 repository migrations at the same time.

#### startRepositoryMigration mutation

mutation startRepositoryMigration (  
  $sourceId: ID\!,  
  $ownerId: ID\!,  
  $repositoryName: String\!,  
  $continueOnError: Boolean\!,  
  $accessToken: String\!,  
  $githubPat: String\!,  
  $gitArchiveUrl: String\!,  
  $metadataArchiveUrl: String\!,  
  $sourceRepositoryUrl: URI\!,  
  $targetRepoVisibility: String\!  
){  
  startRepositoryMigration( input: {  
    sourceId: $sourceId,  
    ownerId: $ownerId,  
    repositoryName: $repositoryName,  
    continueOnError: $continueOnError,  
    accessToken: $accessToken,  
    githubPat: $githubPat,  
    targetRepoVisibility: $targetRepoVisibility  
    gitArchiveUrl: $gitArchiveUrl,  
    metadataArchiveUrl: $metadataArchiveUrl,  
    sourceRepositoryUrl: $sourceRepositoryUrl,  
  }) {  
    repositoryMigration {  
      id  
      migrationSource {  
        id  
        name  
        type  
      }  
      sourceUrl  
    }  
  }  
}

| Query variable | Description |
| :---- | :---- |
| sourceId | Your migration source id returned from the createMigrationSource mutation. |
| ownerId | The organization ID of your organization on GitHub Enterprise Cloud. |
| repositoryName | A custom unique repository name not currently used by any of your repositories owned by the organization on GitHub Enterprise Cloud. An error-logging issue will be created in this repository when your migration is complete or has stopped. |
| continueOnError | Migration setting that allows the migration to continue when encountering errors that don't cause the migration to fail. Must be true or false. We highly recommend setting continueOnError to true so that your migration will continue unless the Importer can't move Git source or the Importer has lost connection and cannot reconnect to complete the migration. |
| githubPat | The personal access token for your destination organization on GitHub Enterprise Cloud. |
| accessToken | The personal access token for your source. |
| targetRepoVisibility | The visibility of the new repository. Must be private, public, or internal. If not set, your repository is migrated as private. |
| gitArchiveUrl | A GitHub Enterprise Cloud-accessible URL for your Git source archive. |
| metadataArchiveUrl | A GitHub Enterprise Cloud-accessible URL for your metadata archive. |
| sourceRepositoryUrl | The URL for your repository on your GitHub Enterprise Server instance. This is required, but GitHub Enterprise Cloud will not communicate directly with your GitHub Enterprise Server instance. |

	  
For personal access token requirements, see Managing access for a migration between GitHub products.

In the next step, you'll use the migration ID returned from the startRepositoryMigration mutation to check the migration status.

### Step 8: Check the status of your migration

To detect any migration failures and ensure your migration is working, you can check your migration status using the getMigration query. You can also check the status of multiple migrations with getMigrations.

The getMigration query will return with a status to let you know if the migration is queued, in progress, failed, or completed. If your migration failed, the Importer will provide a reason for the failure.

#### getMigration query

query (  
  $id: ID\!  
){  
  node( id: $id ) {  
    ... on Migration {  
      id  
      sourceUrl  
      migrationSource {  
        name  
      }  
      state  
      failureReason  
    }  
  }  
}

| Query variable | Description |
| :---- | :---- |
| id | The id of your migration that the startRepositoryMigration mutation returned. |

	

### Step 9: Validate your migration and check the error log

To finish your migration, we recommend that you check the "Migration Log" issue. This issue is created on GitHub in the destination repository.

Screenshot of an issue with the title "Migration Log." The second comment in the issue includes logs for a migration.

Finally, we recommend that you review your migrated repositories for a soundness check.

## GitHub CLI

In this article  
About repository migrations with GitHub Enterprise Importer  
Prerequisites  
Step 1: Install the GEI extension of the GitHub CLI  
Step 2: Update the GEI extension of the GitHub CLI  
Step 3: Set environment variables  
Step 4: Set up blob storage  
Step 5: Generate a migration script  
Step 6: Migrate repositories  
Step 7: Validate your migration and check the error log

### About repository migrations with GitHub Enterprise Importer

You can run your migration with either the GitHub CLI or the API.

The GitHub CLI simplifies the migration process and is recommended for most customers. Advanced customers with heavy customization needs can use the API to build their own integrations with GitHub Enterprise Importer.

To see instructions for using the API, use the tool switcher at the top of the page.

### Prerequisites

* We strongly recommend that you perform a trial run of your migration and complete your production migration soon after. To learn more about trial runs, see Overview of a migration between GitHub products.  
* Ensure you understand the data that will be migrated and the known support limitations of the Importer. For more information, see About migrations between GitHub products.  
* While not required, we recommend halting work during your production migration. The Importer doesn't support delta migrations, so any changes that happen during the migration will not migrate. If you choose not to halt work during your production migration, you'll need to manually migrate these changes.  
* In both the source and destination organizations, you must be either an organization owner or be granted the migrator role. For more information, see Managing access for a migration between GitHub products.  
* If you use GitHub Enterprise Server 3.8 or higher, to configure blob storage for exported archives, you need access to the Management Console.

### Step 1: Install the GEI extension of the GitHub CLI

If this is your first migration, you'll need to install the GEI extension of the GitHub CLI. For more information about the GitHub CLI, see About GitHub CLI.

Alternatively, you can download a standalone binary from the releases page for the github/gh-gei repository. You can run the binary directly, without the gh prefix.

1. Install the GitHub CLI. For installation instructions for GitHub CLI, see the GitHub CLI repository.

Note

You need version 2.4.0 or newer of GitHub CLI. You can check the version you have installed with the gh \--version command.

2. Install the GEI extension.

| Shell |
| :---- |
| gh extension install github/gh-gei |

Any time you need help with the GEI extension, you can use the \--help flag with a command. For example, gh gei \--help will list all the available commands, and gh gei migrate-repo \--help will list all the options available for the migrate-repo command.

### Step 2: Update the GEI extension of the GitHub CLI

The GEI extension is updated weekly. To make sure you're using the latest version, update the extension.

| gh extension upgrade github/gh-gei |
| :---- |

### Step 3: Set environment variables

Before you can use the GEI extension to migrate to GitHub Enterprise Cloud, you must create personal access tokens that can access the source and destination organizations, then set the personal access tokens as environment variables.

1. Create and record a personal access token (classic) that will authenticate for the destination organization on GitHub Enterprise Cloud, making sure that the token meets all requirements. For more information, see Managing access for a migration between GitHub products.

2. Create and record a personal access token that will authenticate for the source organization, making sure that this token also meets all of the same requirements.

3. Set environment variables for the personal access tokens, replacing TOKEN in the commands below with the personal access tokens you recorded above. Use GH\_PAT for the destination organization and GH\_SOURCE\_PAT for the source organization.  
   1. If you're using Terminal, use the export command.

| Shell |
| :---- |
| export GH\_PAT="TOKEN" |
| export GH\_SOURCE\_PAT="TOKEN" |

   2. If you're using PowerShell, use the $env command.

| Shell |
| :---- |
| $env:GH\_PAT="TOKEN" |
| $env:GH\_SOURCE\_PAT="TOKEN" |

4. If you're migrating to GitHub Enterprise Cloud with data residency, for convenience, set an environment variable for the base API URL for your enterprise. For example:

| Shell |
| :---- |
| export TARGET\_API\_URL="https://api.octocorp.ghe.com" |

You'll use this variable with the \--target-api-url option in commands you run with the GitHub CLI.

### Step 4: Set up blob storage

Because many GitHub Enterprise Server instances sit behind firewalls, we use blob storage as an intermediate location to store your data that GitHub can access.

First, you must set up blob storage with a supported cloud provider. Then, you must configure your credentials for the storage provider in the Management Console or GitHub CLI.

#### Setting up blob storage with a supported cloud provider

The GitHub CLI supports the following blob storage providers:

* Amazon Web Services (AWS) S3  
* Azure Blob Storage

##### Setting up an AWS S3 storage bucket

In AWS, set up a S3 bucket. For more information, see Creating a bucket in the AWS documentation.

You will also need an AWS access key and secret key with the following permissions:

{  
    "Version": "2012-10-17",  
    "Statement": \[  
        {  
            "Sid": "VisualEditor0",  
            "Effect": "Allow",  
            "Action": \[  
                "s3:PutObject",  
                "s3:GetObject",  
                "s3:ListBucketMultipartUploads",  
                "s3:AbortMultipartUpload",  
                "s3:ListBucket",  
                "s3:DeleteObject",  
                "s3:ListMultipartUploadParts"  
            \],  
            "Resource": \[  
                "arn:aws:s3:::github-migration-bucket",  
                "arn:aws:s3:::github-migration-bucket/\*"  
            \]  
        }  
    \]  
}  
Note

GitHub Enterprise Importer does not delete your archive from AWS after your migration is finished. To reduce storage costs, we recommend configuring auto-deletion of your archive after a period of time. For more information, see Setting lifecycle configuration on a bucket in the AWS documentation.

##### Setting up an Azure Blob Storage storage account

In Azure, create a storage account and make a note of your connection string. For more information, see Manage storage account access keys in Microsoft Docs.

Note

GitHub Enterprise Importer does not delete your archive from Azure Blob Storage after your migration is finished. To reduce storage costs, we recommend configuring auto-deletion of your archive after a period of time. For more information, see Optimize costs by automatically managing the data lifecycle in Microsoft Docs.

#### Configuring your blob storage credentials

After you set up blob storage with a supported cloud provider, you must configure your credentials for the storage provider in GitHub:

* If you use GitHub Enterprise Server 3.8 or higher, configure your credentials in the Management Console.  
* If you use GitHub Enterprise Server 3.7 or lower, configure the credentials in the GitHub CLI.

##### Configuring blob storage in the Management Console of your GitHub Enterprise Server instance

Note

You only need to configure blob storage in the Management Console if you use GitHub Enterprise Server 3.8 or higher. If you use 3.7 or lower, configure your credentials in the GitHub CLI instead.

After you set up an AWS S3 storage bucket or Azure Blob Storage storage account, configure blob storage in the Management Console of your GitHub Enterprise Server instance. For more information about the Management Console, see Administering your instance from the Management Console.

1. From an administrative account on GitHub Enterprise Server, in the upper-right corner of any page, click .

2. If you're not already on the "Site admin" page, in the upper-left corner, click Site admin.

3. In the " Site admin" sidebar, click Management Console.

4. Log into the Management Console.

5. In the top navigation bar, click Settings.

6. Under Migrations, click Enable GitHub Migrations.

7. Optionally, to import storage settings you configured for GitHub Actions, select Copy Storage settings from Actions. For more information see, Enabling GitHub Actions with Azure Blob storage and Enabling GitHub Actions with Amazon S3 storage.

Note

After copying your storage settings, you may still need to update the configuration of your cloud storage account to work with GitHub Enterprise Importer. In particular, you must ensure that GitHub's IP addresses are allowlisted. For more information, see Managing access for a migration between GitHub products.

8. If you do not import storage settings from GitHub Actions, select either Azure Blob Storage or Amazon S3 and fill in the required details.

Note

If you use Amazon S3, you must set the "AWS Service URL" to the standard endpoint for the AWS region where your bucket is located. For example, if your bucket is located in the eu-west-1 region, the "AWS Service URL" would be https://s3.eu-west-1.amazonaws.com. Your GitHub Enterprise Server instance's network must allow access to this host. Gateway endpoints are not supported, such as bucket.vpce-0e25b8cdd720f900e-argc85vg.s3.eu-west-1.vpce.amazonaws.com. For more information about gateway endpoints, see Gateway endpoints for Amazon S3 in the AWS documentation.

9. Click Test storage settings.

10. Click Save settings.

#### Configuring your blob storage credentials in the GitHub CLI

Note

You only need to configure your blob storage credentials in the GitHub CLI if you use GitHub Enterprise Server 3.7 or lower. If you use 3.8 or higher, configure blob storage in the Management Console instead.

If you configure your blob storage credentials in the GitHub CLI, you will not be able to perform migrations where your Git source or metadata exports exceed 2GB. To perform these migrations, upgrade to GitHub Enterprise Server 3.8 or higher.

##### Configuring AWS S3 credentials in the GitHub CLI

When you're ready to run your migration, you will need to provide your AWS credentials to the GitHub CLI: region, access key, secret key, and session token (if required). You can pass them as arguments, or set environment variables called AWS\_REGION, AWS\_ACCESS\_KEY\_ID, AWS\_SECRET\_ACCESS\_KEY, and AWS\_SESSION\_TOKEN.

You will also need to pass in the name of the S3 bucket using the \--aws-bucket-name argument.

##### Configuring Azure Blob Storage account credentials in the GitHub CLI

When you're ready to run your migration, you can either pass your connection string into the GitHub CLI as an argument, or pass it in using an environment variable called AZURE\_STORAGE\_CONNECTION\_STRING.

#### Allowing network access

If you have configured firewall rules on your storage account, ensure you have allowed access to the IP ranges for your migration destination. See Managing access for a migration between GitHub products.

### Step 5: Generate a migration script

If you want to migrate multiple repositories to GitHub Enterprise Cloud at once, use the GitHub CLI to generate a migration script. The resulting script will contain a list of migration commands, one per repository.

Note

Generating a script outputs a PowerShell script. If you're using Terminal, you will need to output the script with the .ps1 file extension and install PowerShell for either Mac or Linux to run it.

If you want to migrate a single repository, skip to the next step.

#### Generating a migration script

You must follow this step from a computer that can access the API for your GitHub Enterprise Server instance. If you can access your instance from the browser, then the computer has the correct access.

To generate a migration script, run the gh gei generate-script command.

For GitHub Enterprise Server 3.8 or later, or if you're using 3.7 or lower with Azure Blob Storage, use the following flags:

| Shell |
| :---- |
| gh gei generate-script \--github-source-org SOURCE \\   \--github-target-org DESTINATION \\   \--output FILENAME \\   \--ghes-api-url GHES-API-URL |

If you're using GitHub Enterprise Server 3.7 or lower with AWS S3, use the following flags:

| Shell |
| :---- |
| gh gei generate-script \--github-source-org SOURCE \\   \--github-target-org DESTINATION \\   \--output FILENAME \\   \--ghes-api-url GHES-API-URL \\   \--aws-bucket-name AWS-BUCKET-NAME |

##### Placeholders

Replace the placeholders in the command above with the following values.

| Placeholder | Value |
| :---- | :---- |
| SOURCE | Name of the source organization |
| DESTINATION | Name of the destination organization |
| FILENAME | A filename for the resulting migration script If you're using Terminal, use a .ps1 file extension as the generated script requires PowerShell to run. You can install PowerShell for Mac or Linux. |
| GHES-API-URL | The URL for your GitHub Enterprise Server instance's API, such as https://myghes.com/api/v3 |
| AWS-BUCKET-NAME	 | The bucket name for your AWS S3 bucket |

##### Additional arguments

| Argument | Description |
| :---- | :---- |
| \--target-api-url TARGET-API-URL | If you're migrating to GHE.com, add \--target-api-url TARGET-API-URL, where TARGET-API-URL is the base API URL for your enterprise's subdomain. For example: https://api.octocorp.ghe.com. |
| \--no-ssl-verify | If your GitHub Enterprise Server instance uses a self-signed or invalid SSL certificate, use the \--no-ssl-verify flag. With this flag, the GitHub CLI will skip verifying the SSL certificate when extracting data from your instance only. All other calls will verify SSL. |
| \--download-migration-logs | Download the migration log for each migrated repository. For more information about migration logs, see Accessing your migration logs for GitHub Enterprise Importer. |

#### Reviewing the migration script

After you generate the script, review the file and, optionally, edit the script.

* If there are any repositories you don't want to migrate, delete or comment out the corresponding lines.  
* If you want any repositories to have a different name in the destination organization, update the value for the corresponding \--target-repo flag.  
* If you want to change the visibility of new repository, update the value for the corresponding \--target-repo-visibility flag. By default, the script sets the same visibility as the source repository.

If your repository has more than 10 GB of releases data, releases cannot be migrated. Use the \--skip-releases flag to migrate the repository without releases.

If you downloaded GEI as a standalone binary rather than as an extension for the GitHub CLI, you will need to update your generated script to run the binary instead of gh gei.

### Step 6: Migrate repositories

You can migrate multiple repositories with a migration script or a single repository with the gh gei migrate-repo command.

When you migrate repositories, the GEI extension of the GitHub CLI performs the following steps:

1. Connects to your GitHub Enterprise Server instance and generates two migration archives per repository, one for the Git source and one for the metadata  
2. Uploads the migration archives to the blob storage provider of your choice  
3. Starts your migration in GitHub Enterprise Cloud, using the URLs of the archives stored with your blob storage provider  
4. Deletes the migration archive from your local machine

#### Migrate multiple repositories

If you're migrating from GitHub Enterprise Server 3.7 or earlier, before you run your script, you must set additional environment variables to authenticate to your blob storage provider.

* For Azure Blob Storage, set AZURE\_STORAGE\_CONNECTION\_STRING to the connection string for your Azure storage account.

Only connection strings using storage account access keys are supported. Connection strings which use shared access signatures (SAS) are not supported. For more information about storage account access keys, see Manage storage account access keys in the Azure documentation.

* For AWS S3, set the following environment variables.  
  * AWS\_ACCESS\_KEY\_ID: The access key id for your bucket  
  * AWS\_SECRET\_ACCESS\_KEY: The secret key for your bucket  
  * AWS\_REGION: The AWS region where your bucket is located  
  * AWS\_SESSION\_TOKEN: The session token, if you're using AWS temporary credentials (see Using temporary credentials with AWS resources in the AWS documentation)

To migrate multiple repositories, run the script you generated above. Replace FILENAME in the commands below with the filename you provided when generating the script.

* If you're using Terminal, use ./.

| Shell |
| :---- |
| ./FILENAME |

* If you're using PowerShell, use .\\.

| Shell |
| :---- |
| .\\FILENAME |

#### Migrate a single repository

You must follow this step from a computer that can access the API for your GitHub Enterprise Server instance. If you can access your instance from the browser, then the computer has the correct access.

To migrate a single repository, use the gh gei migrate-repo command.

If you're using GitHub Enterprise Server 3.8 or later, use the following flags:

| Shell |
| :---- |
| gh gei migrate-repo \--github-source-org SOURCE \--source-repo CURRENT-NAME \--github-target-org DESTINATION \--target-repo NEW-NAME \--ghes-api-url GHES-API-URL |

If you're migrating from GitHub Enterprise Server 3.7 or earlier and using Azure Blob Storage as your blob storage provider, use the following flags to authenticate:

| Shell |
| :---- |
| gh gei migrate-repo \--github-source-org SOURCE \--source-repo CURRENT-NAME \--github-target-org DESTINATION \--target-repo NEW-NAME \\     \--ghes-api-url GHES-API-URL \--azure-storage-connection-string "AZURE\_STORAGE\_CONNECTION\_STRING" |

If you're migrating from GitHub Enterprise Server 3.7 or earlier and using Amazon S3 as your blob storage provider, use the following flags to authenticate:

| Shell |
| :---- |
| gh gei migrate-repo \--github-source-org SOURCE \--source-repo CURRENT-NAME \--github-target-org DESTINATION \--target-repo NEW-NAME \\     \--ghes-api-url GHES-API-URL \--aws-bucket-name "AWS-BUCKET-NAME" |

##### Placeholders

Replace the placeholders in the command above with the following values.

| Placeholder | Value |
| :---- | :---- |
| SOURCE | Name of the source organization |
| CURRENT-NAME | The name of the repository you want to migrate |
| DESTINATION | Name of the destination organization |
| NEW-NAME | The name you want the migrated repository to have |
| GHES-API-URL | The URL for your GitHub Enterprise Server instance's API, such as https://myghes.com/api/v3 |
| AZURE\_STORAGE\_CONNECTION\_STRING | The connection string for your Azure storage account. Make sure to quote the connection string, which contains characters that would likely otherwise be interpreted by shell. |
| AWS-BUCKET-NAME | The bucket name for your AWS S3 bucket |

##### Additional arguments

| Argument | Description |
| :---- | :---- |
| \--target-api-url TARGET-API-URL | If you're migrating to GHE.com, add \--target-api-url TARGET-API-URL, where TARGET-API-URL is the base API URL for your enterprise's subdomain. For example: https://api.octocorp.ghe.com.  |
| \--no-ssl-verify | If your GitHub Enterprise Server instance uses a self-signed or invalid SSL certificate, use the \--no-ssl-verify flag. With this flag, the GitHub CLI will skip verifying the SSL certificate when extracting data from your instance only. All other calls will verify SSL. |
| \--skip-releases | If your repository has more than 10 GB of releases data, releases cannot be migrated. Use the \--skip-releases flag to migrate the repository without releases. |
| \--target-repo-visibility TARGET-VISIBILITY | Repositories are migrated with private visibility by default. To set the visibility, you can add the \--target-repo-visibility flag, specifying private, public, or internal. If you're migrating to an enterprise with managed users, public repositories are not available. |

	

##### Aborting the migration

If you want to cancel a migration, use the abort-migration command, replacing MIGRATION-ID with the ID returned from migrate-repo.

| Shell |
| :---- |
| gh gei abort-migration \--migration-id MIGRATION-ID |

### Step 7: Validate your migration and check the error log

When your migration is complete, we recommend reviewing your migration log. For more information, see Accessing your migration logs for GitHub Enterprise Importer.

We recommend that you review your migrated repositories for a soundness check.

After your migration has finished, we recommend deleting the archives from your storage container. If you plan to complete additional migrations, delete the archive placed into your storage container by the ADO2GH extension. If you're done migrating, you can delete the entire container.

# About GitHub Enterprise Importer

With GitHub Enterprise Importer, you can migrate your enterprise to GitHub Enterprise Cloud from various sources.

In this article  
About GitHub Enterprise Importer  
Supported migration paths  
Getting started  
About GitHub Enterprise Importer  
GitHub Enterprise Importer is a highly customizable migration tool designed to help you move your enterprise to GitHub Enterprise Cloud.

You can migrate on a repository-by-repository basis or, if your migration source is GitHub.com, on an organization-by-organization basis.

GitHub Enterprise Importer allows you to customize your migration to meet your enterprise's unique needs with:

A distinct migration permissions role for repository migrations, which allows you to designate teams and/or individual users to run a migration and removes the need for organization owners to complete the migration.  
High fidelity migration, which allows you to migrate a single repository, a series of repositories, or an entire organization.  
Support for custom trial run migrations, which allow you to run a migration as many times as you desire before running the production migration.  
Clear and unblocking error logging, so that migrations are allowed to continue with non-critical migration errors, such as not being able to move a single pull request comment. After the migration, you can review a log file that opens automatically.  
Users retain ownership of their history, to ensure that their Git history or GitHub metadata is maintained across the migration.  
You can run your migration with either the GitHub CLI or the API.

The GitHub CLI simplifies the migration process and is recommended for most customers. Advanced customers with heavy customization needs can use the API to build their own integrations with GitHub Enterprise Importer.

Supported migration paths  
GitHub Enterprise Importer supports migrations to GitHub Enterprise Cloud (GitHub.com or GHE.com) from the following sources.

Azure DevOps (ADO) Cloud  
Bitbucket Server and Bitbucket Data Center 5.14+  
GitHub.com  
GitHub Enterprise Server (GHES) 3.4.1+

