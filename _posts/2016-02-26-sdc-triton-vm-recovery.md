---
title: "Howto recover a vm in sdc/triton"
date: 2016-02-26 15:54:00
categories: sdc triton vm recover vmapi vmadm 
---

Last night I've attempted an upgrade to an SDC/Triton cloud environment where the headnode had some custom backup scripts.

*NOTE: if you are using zfs snapshorts to backup the headnode don't reserve them!*

So after I start upgrade process I start receiving errors like this:
{% highlight shell %}
--- Updating cnapi ...
Installing image 4559deda-d66c-11e5-a487-ff3bb057a2cb
    (cnapi@release-20160218-20160218T181620Z-ge02f68c)
Reprovisioning cnapi VM 1c0f6702-467c-499c-872c-1347e2fa4442
Update error: {"message":"error reprovisioning VM 1c0f6702-467c-499c-872c-1347e2fa4442: exit code 1, signal null\n    stdout:\n            stderr:\n        Failed to reprovision VM 1c0f6702-467c-499c-872c-1347e2fa4442: Command failed: cannot destroy snapshot zones/1c0f6702-467c-499c-872c-1347e2fa4442-reprovisioning-root@backup-1456455300: dataset is busy\n        cannot destroy snapshot zones/1c0f6702-467c-499c-872c-1347e2fa4442-reprovisioning-root@backup-1456455300: dataset is busy\n        \n        ","code":"InternalError","exitStatus":1}
sdcadm update: error: error reprovisioning VM 1c0f6702-467c-499c-872c-1347e2fa4442: exit code 1, signal null
    stdout:
            stderr:
        Failed to reprovision VM 1c0f6702-467c-499c-872c-1347e2fa4442: Command failed: cannot destroy snapshot zones/1c0f6702-467c-499c-872c-1347e2fa4442-reprovisioning-root@backup-1456455300: dataset is busy
        cannot destroy snapshot zones/1c0f6702-467c-499c-872c-1347e2fa4442-reprovisioning-root@backup-1456455300: dataset is busy
{% endhighlight %}

To recover from this first you have to remove the snapshot first:
{% highlight shell %}
$ zfs holds  zones/1c0f6702-467c-499c-872c-1347e2fa4442@backup-1456455300
NAME                                                                TAG    TIMESTAMP
zones/1c0f6702-467c-499c-872c-1347e2fa4442@backup-1456455300  backup Thu Feb 26 11:23:45 2016

$ zfs release -r backup zones/1c0f6702-467c-499c-872c-1347e2fa4442@backup-1456455300

$ zfs destroy -rF  zones/1c0f6702-467c-499c-872c-1347e2fa4442@backup-1456455300

{% endhighlight %}

And then, because the upgrade process has stop in the middle, you have to remove the reprovision image

{% highlight shell %}
$ zfs destroy -rF  zones/1c0f6702-467c-499c-872c-1347e2fa4442-reprovisioning-root
{% endhighlight %}

and then restart the upgrade process (after you removed the lock file)
{% highlight shell %}
$ sdcadm upgrade --all
{% endhighlight %}

The interesting thing happen with docker vm instance that refused to be upgraded anymore, it was in an invalid state.

{% highlight shell %}
$ vmadm list | grep docker
bb1aefde-444e-45f4-aa7b-9fb717883e23  OS    4096     invalid           docker0
{% endhighlight %}

No matter what I've tried it could not be recoverd or restarted with the usual sdc tools. The error was either the instance has to be running to perform an upgrade or the instance has to be stopped to be recovered. Therefore I've applyed the following procedure to recover it.

First I've created a file with the image json data.
{% highlight shell %}
$ sdc-vmadm get bb1aefde-444e-45f4-aa7b-9fb717883e23 > file.json
{% endhighlight %}

Then I've recreate the vm using the SmartOS tools (not SDC TOOLS!)
{% highlight shell %}
$ vmadm create < file.json
{% endhighlight %}

Then you force vmapi to update the vm state as it's exaplained [here](https://docs.joyent.com/private-cloud/troubleshooting/force-vmapi-sync)
{% highlight shell %}
$ sdc sdc-vmapi /vms/bb1aefde-444e-45f4-aa7b-9fb717883e23?sync=true 
{% endhighlight %}

And finally you can restart the upgrade process. To be noted here, because I did'nt have any running instances in docker in that moment I can't confirm if the status of the running docker vms has been preserved or not.






