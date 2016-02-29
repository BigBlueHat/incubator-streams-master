Release Process


Release Steps Overview
There are two distinct sets of artifacts that are released on independent schedules: streams-master & streams-project. The streams-master is the project metadata and only needs to be released when there is a change in the structure of the project itself. The streams-project artifacts are comprised of all streams source code, binaries and a standalone demo. For release setup information, refer to Release Setup Information.

All of the steps below apply to both the master and project releases, unless otherwise specified. As an alternative to releasing separately, the projects MAY be released together as one combined release. The steps for this can be found below. (Combined Release Steps)

Common Release Steps

Environment setup for releasing artifacts (same for SNAPSHOTs and releases)

Increase the default Java heap available to Maven (required for Java SE 6)

export MAVEN_OPTS="-Xmx1024m -XX:MaxPermSize=256m"

Use the latest Sun 1.7.0 JDK

Use Maven 3.2.1 or later

Make sure the Release Setup steps have been performed.

Prepare the source for release:

Cleanup JIRA so the Fix Version in issues resolved since the last release includes this release version correctly.
Update the text files in a working copy of the project root -
Update the CHANGELOG based on the Text release reports from JIRA.
Review and update README.md if needed.
Commit any changes back to git
Stage any Roadmap or Release landing pages on the site.
Create a release candidate branch from master. X should start at 1 and increment if early release candidates fail to complete the release cycle.

$ git checkout master $ git branch {$project.version}-rcX

Verify the source has the required license headers before trying to release:

$ mvn -P apache-release clean rat:check -e -DskipTests

Do a dry run of the release:prepare step:

$ mvn -P apache-release release:prepare -DautoVersionSubmodules=true -DdryRun=true
The dry run will not commit any changes back to SCM and gives you the opportunity to verify that the release process will complete as expected. You will be prompted for the following information :

Release version - take the default (should be ${project.version}-incubating)
SCM release tag - DO NOT TAKE THE DEFAULT - ${project.version}-rcX
New development version - take the default (should be ${project.version}-incubating-SNAPSHOT)
GPG Passphrase
If you cancel a release:prepare before it updates the pom.xml versions, then use the release:clean goal to just remove the extra files that were created.

The Maven release plugin checks for SNAPSHOT dependencies in pom's. It will not complete the prepare goal until all SNAPSHOT dependencies are resolved.

Verify that the release process completed as expected

The release plugin will create pom.xml.tag files which contain the changes that would have been committed to SVN. The only differences between pom.xml.tag and it's corresponding pom.xml file should be the version number.
Check release.properties and make sure that the scm properties have the right version. Sometimes the scm location can be the previous version not the next version.
Verify signatures (Verifying release signatures)
Cleanup the release prepare files again:

$ mvn -P apache-release release:clean
Prepare the release

Run the "release:prepare" step for real this time. You'll be prompted for the same version information.

$ mvn -P apache-release -U clean release:prepare -DautoVersionSubmodules=true

Backup (zip or tar) your local release candidate directory in case you need to rollback the release after the next step is performed.

Perform the release

This step will create a maven staging repository and site for use in testing and voting.

$ mvn -Papache-release -Darguments='-Dmaven.test.skip.exec=true' release:perform -Dgoals=deploy -DlocalRepoDirectory=. -DlocalCheckout=true

If your local OS userid doesn't match your Apache userid, then you'll have to also override the value provided by the OS to Maven for the site-deploy step to work. This is known to work for Linux, but not for Mac and unknown for Windows.

-Duser.name=[your_apache_uid]

Verify the Nexus release artifacts

Verify the staged artifacts in the nexus repo

https://repository.apache.org/index.html
Staging repositories (under Build Promotion) --> Name column --> org.apache.streams
Navigate through the artifact tree and make sure that all javadoc, sources, tests, jars, ... have .asc (GPG signature) and .md5 files. See http://people.apache.org/~henkp/repo/faq.html and http://www.apache.org/dev/release-signing.html#openpgp-ascii-detach-sig
Close the nexus staging repo

https://repository.apache.org/index.html
Staging repositories (under Build Promotion) --> Name column --> org.apache.streams
Click checkbox for the open staging repo (org.apache.streams-XXX) and press Close in the menu bar.
Put the release candidate up for a vote

Create a VOTE email thread on dev@ to record votes as replies
Create a DISCUSS email thread on dev@ for any vote questions
Perform a review of the release and cast your vote. See the following for more details on Apache releases

http://www.apache.org/dev/release.html

A -1 vote does not necessarily mean that the vote must be redone, however it is usually a good idea to rollback the release if a -1 vote is received. See - Recovering from a vetoed release

After the vote has been open for at least 72 hours, has at least three +1 PMC votes and no -1 votes, then post the results to the vote thread by -
reply to the initial email and prepend to the original subject "[RESULT]"
Include a list of everyone who voted +1, 0 or -1.
If there are fewer than 3 +1 (binding) votes from IPMC members, submit a vote to general@incubator.apache.org requesting additional IPMC member votes.
Finalizing a release

Promote the staged nexus artifacts

https://repository.apache.org/index.html
Staging repositories (under Build Promotion) --> Name column --> org.apache.streams
Click checkbox of the closed staging repo (org.apache.streams-XXX) and select Release.
Copy the source artifacts over to the distribution area

$ svn co https://dist.apache.org/repos/dist/release/incubator/streams/releases ./streams-releases (KEEP this directory until after the release process has been completed)
$ cd ./streams-releases
$ mkdir ${project.version}
$ cd ./${project.version}
$ wget https://repository.apache.org/content/repositories/releases/org/apache/streams/${project.name}/${project.version}/${project.name}-${project.version}-source-release.zip
$ wget https://repository.apache.org/content/repositories/releases/org/apache/streams/${project.name}/${project.version}/${project.name}-${project.version}-source-release.zip.asc
$ wget https://repository.apache.org/content/repositories/releases/org/apache/streams/${project.name}/${project.version}/${project.name}-${project.version}-source-release.zip.md5
$ svn add ${project.name}-*
$ svn commit -m "Committing Source Release for ${project.name}-${project.version}
Create an official release tag from the successful release candidate tag.

$ git checkout streams-project-${project.version}-rcX
$ git tag -a streams-project-${project.version} -m 'release tag streams-project-${project.version}'
$ git push origin :refs/tags/streams-project-${project.version}
Update the staged website

Update the downloads page to add new version using the mirrored URLs
Modify the URL for the prior release to the archived URL for the release
Publish the website

WAIT 24hrs after committing releases for mirrors to replicate
Publish updates to the download page
Delete the prior versions

Navigate to the release directories checked out in the prior steps
Delete the prior release artifacts using the svn delete command
Commit the deletion
Update the JIRA versions page to close all issues, mark the version as "released", and set the date to the date that the release was approved. You may also need to make a new release entry for the next release.

Announcing the release

Make a news announcement on the streams homepage.
Make an announcement about the release on the users@streams.apache.org, dev@streams.apache.org, and announce@apache.org list as per the Apache Announcement Mailing Lists page)
Recovering from a vetoed release

Reply to the initial vote email and prepend to the original subject -

[CANCELED]

Clean the release prepare files and hard reset the release candidate branch.

mvn -P apache-release release:clean

Delete the git tag created by the release:perform step -

$ git tag -d streams-project-${project.version}-rcX $ git push origin :refs/tags/streams-project-${project.version}-rcX

Delete the build artifacts on people & www

$ rm -rfv /www/people.apache.org/builds/streams/${project.version}
$ rm -rfv /www/www.apache.org/dist/streams/${project.version}
Drop the nexus staging repo

https://repository.apache.org/index.html
Enterprise --> Staging
Staging tab --> Name column --> org.apache.streams
Right click on the closed staging repo (org.apache.streams-XXX) and select Drop.
Remove the staged site

Make the required updates that caused the vote to be canceled during the next release cycle


Verifying release signatures

On unix platforms the following command can be executed -

  for file in `find . -type f -iname '*.asc'`
  do
      gpg --verify ${file}
  done
You'll need to look at the output to ensure it contains only good signatures -

gpg: Good signature from ... gpg: Signature made ...


Combined Release

In order to perform a combined release of the streams-master and streams-project trunks, do the following:

Perform Steps 1-9 of the release for streams Master & streams Project
Do NOT perform step 10 until steps 1-9 have been completed for BOTH projects
Build the streams-master FIRST
When prompted to change the streams-project dependency on streams-master SNAPSHOT, do so to the release that you just built
Execute the remaining steps using the following e-mail templates
PMC Release Vote