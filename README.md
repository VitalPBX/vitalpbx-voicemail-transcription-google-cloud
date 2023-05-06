# Send the Voicemail with a transcription with Google Cloud
Next we show the steps to follow to send the Voicemail with the transcription of the content. There are two ways to do it, using the Script or doing it step by step.

## Prerequisites
- Have installed VitalPBX 4 and configured the sending of Email. This is done in ADMIN/System Settings/Email Settings.
- Create an account, if you don't have yet, with Google Cloud. Free trials can be made at https://console.cloud.google.com/freetrial
- Within console.cloud.google.com search for Cloud Speech-to-Text API and enable it
- Have root access to the VitalPBX Server

Install Required Dependencies for Google Cloud ClI
<pre>
apt install curl apt-transport-https gnupg jq sox flac dos2unix gnupg
</pre>

## Install Google Cloud CLI
You can acquire the public Key for Google Cloud by executing the following command.
<pre>
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
</pre>

Use the following command to add the gcloud CLI distribution URI as a packet source.
<pre>
echo "deb https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
</pre>

Run the following command to update and install the gcloud CLI.
<pre>
apt-get update && sudo apt-get install google-cloud-cli
</pre>

Use the following command to initialize the gcloud CLI.
<pre>
gcloud init
</pre>

Now we are going to authorize the Asterisk user to also have access to Google Cloud.
We change user in the Linux console.
<pre>
su -s /bin/bash asterisk
</pre>

We now request access to Google Cloud.
<pre>
gcloud auth login
</pre>

Finally we do the test and it should give us the same result as before.
<pre>
gcloud ml speech recognize 'gs://cloud-samples-tests/speech/brooklyn.flac' --language-code='en-US'
</pre>

Now we download the sendmail-gcloud file and copy it to /usr/sbin/
<pre>
wget https://raw.githubusercontent.com/VitalPBX/vitalpbx-voicemail-transcription-google-cloud/main/sendmail-gcloud /usr/sbin/
</pre>

Later we create the file voicemail__60-general.conf in /etc/Asterisk/vitalpbx/ with the following content.
<pre>
nano /etc/asterisk/vitalpbx/voicemail__60-general.conf 
</pre>

Content
<pre>
[general](+)
;You override the default program to send e-mail to use the script
mailcmd=/usr/sbin/sendmail-gcloud
</pre>

Now we assign the corresponding permissions
<pre>
cd /usr/sbin/
chown asterisk:asterisk sendmail-gcloud
chmod 744 sendmail-gcloud
chmod 777 /usr/bin/dos2unix
</pre>

Finally we do a reload of the voicemail module
<pre>
asterisk -rx"module reload app_voicemail.so"
</pre>

Overall, voicemail transcription can be a useful tool for anyone who receives voicemails regularly, and it can provide several benefits, including time savings, improved accessibility, increased accuracy, and convenience.

For more details about the implementation of Google Cloud VoiceMail Transcription, visit the following Blog:

https://vitalpbx.com/blog/vitalpbx-voicemail-transcription-with-google/
