---
description: Describes how to use the Amazon CloudWatch Logs logging driver.
keywords: AWS, Amazon, CloudWatch, logging, driver
redirect_from:
- /engine/reference/logging/awslogs/
title: Amazon CloudWatch Logs logging driver
---

The `awslogs` logging driver sends container logs to
[Amazon CloudWatch Logs](https://aws.amazon.com/cloudwatch/details/#log-monitoring).
Log entries can be retrieved through the [AWS Management
Console](https://console.aws.amazon.com/cloudwatch/home#logs:) or the [AWS SDKs
and Command Line Tools](http://docs.aws.amazon.com/cli/latest/reference/logs/index.html).

## Usage

You can configure the default logging driver by passing the `--log-driver`
option to the Docker daemon:

    dockerd --log-driver=awslogs

You can set the logging driver for a specific container by using the
`--log-driver` option to `docker run`:

    docker run --log-driver=awslogs ...

## Amazon CloudWatch Logs options

You can use the `--log-opt NAME=VALUE` flag to specify Amazon CloudWatch Logs logging driver options.

### awslogs-region

The `awslogs` logging driver sends your Docker logs to a specific region. Use
the `awslogs-region` log option or the `AWS_REGION` environment variable to set
the region.  By default, if your Docker daemon is running on an EC2 instance
and no region is set, the driver uses the instance's region.

    docker run --log-driver=awslogs --log-opt awslogs-region=us-east-1 ...

### awslogs-group

You must specify a
[log group](http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/WhatIsCloudWatchLogs.html)
for the `awslogs` logging driver.  You can specify the log group with the
`awslogs-group` log option:

    docker run --log-driver=awslogs --log-opt awslogs-region=us-east-1 --log-opt awslogs-group=myLogGroup ...

### awslogs-stream

To configure which
[log stream](http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/WhatIsCloudWatchLogs.html)
should be used, you can specify the `awslogs-stream` log option.  If not
specified, the container ID is used as the log stream.

> **Note**:
> Log streams within a given log group should only be used by one container
> at a time.  Using the same log stream for multiple containers concurrently
> can cause reduced logging performance.

### awslogs-create-group

{% include edge_only.md section="option" %}

Log driver will return an error by default if the log group does not exist. However, you can set the
`awslogs-create-group` to `true` to automatically create the log group as needed.
The `awslogs-create-group` option defaults to `false`.

```bash
$ docker run --log-driver=awslogs \
             --log-opt awslogs-region=us-east-1 \
             --log-opt awslogs-group=myLogGroup \
             --log-opt awslogs-create-group=true \
             ...
```

> **Note**:
> Your AWS IAM policy must include the `logs:CreateLogGroup` permission before you attempt to use `awslogs-create-group`.


### tag

Specify `tag` as an alternative to the `awslogs-stream` option. `tag` interprets template markup (e.g., `{% raw %}{{.ID}}{% endraw %}`, `{% raw %}{{.FullID}}{% endraw %}` or `{% raw %}{{.Name}}{% endraw %}` `{% raw %}docker.{{.ID}}{% endraw %}`).
See the [tag option documentation](log_tags.md) for details on all supported template substitutions.

When both `awslogs-stream` and `tag` are specified, the value supplied for `awslogs-stream` will override the template specified with `tag`.

If not specified, the container ID is used as the log stream.

{% raw %}
> **Note**:
> The CloudWatch log API doesn't support `:` in the log name. This can cause some issues when using the `{{ .ImageName }}` as a tag, since a docker image has a format of `IMAGE:TAG`, such as `alpine:latest`.
> Template markup can be used to get the proper format. 
> To get the image name and the first 12 characters of the container id, you can use: `--log-opt tag='{{ with split .ImageName ":" }}{{join . "_"}}{{end}}-{{.ID}}'`
> the output will be something like: `alpine_latest-bf0072049c76` 
{% endraw %}


## Credentials

You must provide AWS credentials to the Docker daemon to use the `awslogs`
logging driver. You can provide these credentials with the `AWS_ACCESS_KEY_ID`,
`AWS_SECRET_ACCESS_KEY`, and `AWS_SESSION_TOKEN` environment variables, the
default AWS shared credentials file (`~/.aws/credentials` of the root user), or
(if you are running the Docker daemon on an Amazon EC2 instance) the Amazon EC2
instance profile.

Credentials must have a policy applied that allows the `logs:CreateLogStream`
and `logs:PutLogEvents` actions, as shown in the following example.

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Action": [
            "logs:CreateLogStream",
            "logs:PutLogEvents"
          ],
          "Effect": "Allow",
          "Resource": "*"
        }
      ]
    }
