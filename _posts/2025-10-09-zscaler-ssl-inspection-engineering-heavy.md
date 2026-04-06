---
layout: post
title: Zscaler SSL Inspection in an Engineering heavy organization
---

# The Problem

My work recently switched to using Zscaler Internet Access (ZIA) and implemented SSL Inspection. For most users, this didn't cause any impact as browser traffic gets handled correctly. However, many users in our Egnineering department (~1/3 of the employees) hit issues when running command line tools or clients outside the browser (think Github Desktop or Visual Studio). These errors occur when a tool performs certificate pinning or detects the MITM Zscaler traffic inspection, and the error will usually have a SSL error like "CERTIFICATE_VERIFY_FAILED" and "failed to verify certificate: x509: certificate signed by unknown authority". 

Get used to seeing the following:
![alt text][certerror]

[certerror]:/public/certerror.png "Certificate Error"

# The Workaround
To workaround this issue, Zscaler recommends bypassing SSL inspection for traffic from affected domains. Zscaler provides many domains in a support article [here](https://help.zscaler.com/zia/certificate-pinning-and-ssl-inspection), but we found that many more domains needed to be SSL inspection bypassed. This process was tedious and felt like categorizing the internet one domain at a time. We manage our SSL bypass domains in Terraform, and I have provided oit below. Hopefully it saves you some time!

# SSL Inspection bypass list

```
.alpinelinux.org
.analytics.google.com
.api.sanity.io
.blob.core.windows.net
.cricut.com
.cursor-cdn.com
.cursor.sh
.cursorapi.com
.datadoghq.com
.dl.k8s.io
.ehawaii.gov
.github.com
.golang.org
.googleapis.com
.hashicorp.com
.hawaii.gov
.microsoft.com
.netlify.app
.netlify.com
.online.visualstudio.com
.pypi.org
.pythonhosted.org
.riverside.fm
.sanity.io
.safebase.io
.sendgrid.net
.terraform.io
.liveshare.vsengsaas.visualstudio.com
.vsassets.io
.vscode-cdn.net
.yarnpkg.com
aetna.login-usb.mimecast.com
aka.ms
api.cloudflare.com
api.figma.com
api.getmontecarlo.com
api.nuget.org
apt.releases.hashicorp.com
artifact-assets-prod-us-east-1.s3.amazonaws.com
artifacts.elastic.co
baltocdn.com
community.chocolatey.org
dl.cloudsmith.io
dl.k8s.io
docker.cloudsmith.io
downloads.microsoft.com
figma-alpha-api.s3.us-west-2.amazonaws.com
figma.com
get.helm.sh
github-cloud.s3.amazonaws.com
github.com
go.dev
hashicorp.com
hub.getdbt.com
intime.dor.in.gov
lwbnlive.accounts.ondemand.com
mcr.microsoft.com
microsoft.com
netlify.app
netlify.com
nodejs.org
np64869.east-us-2.azure.snowflakecomputing.com
nuget.cloudsmith.io
api.openai.com
packages.cloud.google.com
packages.microsoft.com
pkgs.k8s.io
pmc-geofence.trafficmanager.net
portal.us.bn.cloud.ariba.com
proxy.golang.org
pypi.org
pythonhosted.org
quickview.cloudapps.cisco.com
registry.npmjs.com
registry.npmjs.org
rekor.sigstore.dev
repo1.maven.org
rootly.com
rubygems.org
service.ariba.com
static-production.npmjs.com
terraform.io
tuf-repo-cdn.sigstore.dev
updates.vc.logitech.com
us-docker.pkg.dev
test.salesforce.com
```