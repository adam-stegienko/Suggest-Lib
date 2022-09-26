# suggest-lib

Tasks:

0. Config your Jenkins, GitLab, and JFrog envs for suggest-lib and push source code to GitLab. -- JFrog OK // push source code OK // Jenkins OK // GitLab OK
1. Add a developer machine to your lab docker compose topology -- PASSED
2. Create a multibranch pipeline. -- PASSED
3. Follow the case of non-release branch: -- PASSED
    a. compile -- PASSED
    b. test -- PASSED
    c. mvn verify -- PASSED
    d. run on developer machine for the purpose of hands-on testing -- NO POSSIBILITY/NO REASON TO DO THAT --> PASSED
4. Follow the case of master branch:
    a. compile
    b. test
    c. mvn deploy
    d. publish as SNAPSHOT in JFrog artifactory
5. Follow the case of release branch:
    a. Calculate the correct version based on branch and latest used tag
    b. compile
    c. test
    d. change version in build (mvn versions:set at mvn install stage)
    e. mvn deploy
    f. publish as $NEW_VERSION in JFrog artifactory
    g. Tag the new version using git
    h. Push the tag and the updated branch to Git repo (GitLab)













TO DO OR USEFUL TIPS FOR THE TASK:
- publishing on master should result in "SNAPSHOT" saved in artifactory
- developer machine as additional container with maven:3.6.2-jdk-8 --> also a volume for ./m2/repository:/.m2/repository, as well as volume on working files (Whatever they are), and a volume on docker socket to trace the developer's efforts from the main instance!
- developer machine can be connected with the rest of servers via one big Docker Compose!
- JF - tools --> JDK and maven
--> params BRANCH, GIT_BRANCH
--> stages, minimum is Build, Test, and Tagging stages
--> you use settings.xml for the build and test
--> use regex for tagging, e.g env.GIT_BRANCH ==~ /release(.+)/
--> remember about double quotes and escaping special characters!
--> For credentials remeber about the single quotes to avoid creds leakage!

Developer Machine: docker run -it --name=dev_server -v ~/DevOps/AdamLab/suggest-lib:/usr/suggest-lib --workdir=/usr/suggest-lib --entrypoint='bash' maven:3.6.2-jdk-8

---------------- IRRELEVANT FOR THE PREVIOUS TASK SPECIFICALLY--------------

TO DO WITH TOXIC TYPO:
- unit tests and e2e tests for both release and branch
- if master branch, additionally deploy to PROD AWS instance
- (Bonus: you can use slaves to make the tests faster)
- For the next Phase B --> use a load balancer from AWS (ALB/ELB) to balance the traffic between two ToxicTypo App instances
- Listener --> configs indicating how both apps instances communicate with ALB
- Catch your load balancer's DNS name and use it the specific port you chose for LB
- PREPARE FOR SHOW AND TELL !!! 