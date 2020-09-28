# Chef Automate Teams Notifications

## Introduction

Greetings! This repo contains a workaround to allow [Chef Automate](https://automate.chef.io) to send failure notifications via email using [Jenkins](https://www.jenkins.io)

## Components

To facilitate the notifications, this repo uses the [Generic Webhook Trigger](https://plugins.jenkins.io/generic-webhook-trigger/) to intercept the webhook from Automate, translate it into variables, then send the data as a formatted email.

## Assumptions

This repository assumes the following:

* You have an existing, working instance of Jenkins.
* The Jenkins instance is configured to send email.
* You know how to use your email plugin of choice (e.g. you can create templated based on pipeline variables).

## Usage

Since we're using Jenkins as a process automation tool and not a CI/CD tool, we need to add an additional trigger as there aren't any branches or merges. To do this, we'll use the [Generic Webhook Trigger](https://plugins.jenkins.io/generic-webhook-trigger/) Jenkins plugin. Once the plugin is installed, we can clone the [Jenkinsfile](./Jenkinsfile) in this repository and tailor our email. Once we've completed our customizations and stored the `Jenkinsfile` in a new repo, we can create a new project in Jenkins.

The Webhook trigger is the real heavy lifter here. As we can see in the code below, the plugin is intercepting the full webhook payload from [Chef Automate](https://automate.chef.io) and assigning the desired bits of data to variable that Jenkins can use.

```groovy
triggers {
  GenericTrigger(genericVariables: [
                [key: 'type', value: '$.type'],
                [key: 'total_number_of_tests', value: '$.total_number_of_tests'],
                [key: 'total_number_of_skipped_tests', value: '$.total_number_of_skipped_tests'],
                [key: 'total_number_of_passed_tests', value: '$.total_number_of_passed_tests'],
                [key: 'total_number_of_failed_tests', value: '$.total_number_of_failed_tests'],
                [key: 'number_of_failed_critical_tests', value: '$.number_of_failed_critical_tests'],
                [key: 'number_of_critical_tests', value: '$.number_of_critical_tests'],
                [key: 'node_name', value: '$.node_name'],
                [key: 'automate_failure_url', value: '$.automate_failure_url']
                ], causeString: '$type on $node_name', regexpFilterExpression: '', regexpFilterText: '', printContributedVariables: true, printPostContent: true, token: '1')
  }
```

From here, we can simply use the variables to format an email. Depending on the email plugin you're using, you can dress up the email quite a bit.

```groovy
steps {
  emailext(subject: '$type for $node_name', body: '$total_number_of_passed_tests / $total_number_of_failed_tests / $total_number_of_skipped_tests / $total_number_of_tests')
}
```

## Customizing

A couple things to note:

* Automate has two separate notification types
  * Converge failures - Data on when chef-client fails
  * Compliance failures - Data when Inspec reports non-compliant findings
* While the example above only shows a few data points, the notification payloads contain quite a bit of information. You can see example payloads in the [Automate Docs](https://docs.chef.io/automate/notifications/#webhook-notification-payload).

## Closing

If all went well, you should start seeing notifications within a few minutes (give or take, depending on your client interval/splay).

Hopefully, you find this to be useful. If you have any questions or feedback, please feel free to contact me: [@jamesmassardo](https://twitter.com/jamesmassardo).