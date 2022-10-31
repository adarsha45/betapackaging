BUILD_NUMBER=env.BUILD_NUMBER
JOB_NAME = env.JOB_NAME
WORKSPACE = env.WORKSPACE
INSTANCE_URL = 'https://login.salesforce.com/'
PACKAGING_ORG_USERNAME = 'empathetic-shark-ckaajb.com';
SOURCE_API_NAME = "53.0"
PACKAGE_NAME = "ComplianceQuest"

node{
    checkout scm;

    // Because bitbucketStatusNotify only works from second build
    if ("${BUILD_NUMBER}" == "1") {
        build job: "${JOB_NAME}", wait: false
        currentBuild.result = 'ABORTED'
        error("Skipping build ${BUILD_NUMBER} as it doesn't play nice with bitbucket notifications")
    }
    // This limits build concurrency to 1 per branch
    properties([disableConcurrentBuilds()])
    

    try{
        // Send build started status to bitbucket
        bitbucketStatusNotify(buildState: 'INPROGRESS');

        stage('Authorize the packaging org'){
            withCredentials([file(credentialsId: '2ae5c09d-e16d-429a-97c4-6d1713aa18e1', variable: 'server_key_file')]) 
                {
                try {
                    sh "sfdx force:auth:logout -u ${PACKAGING_ORG_USERNAME} --noprompt"
                } catch (Exception e) {
                    //continue if no org exists to logout
                }
                sh "sfdx auth:jwt:grant --instanceurl ${INSTANCE_URL} --clientid ${cq_consumer_secret} --username ${PACKAGING_ORG_USERNAME} --jwtkeyfile ${jwt_key_file} --setalias ${PACKAGING_ORG_USERNAME}";
            }
        }

        // stage('Build the Source Code'){
        //     dir ('build/antscripts') {
        //         sh("ant -f project.build.xml -D'dep.sf.username=${PACKAGING_ORG_USERNAME}' initializeproperties packageresource");
        //     }
        // }

        // stage('Create Delta Package'){
        //     def response = sh(returnStdout: true, script: "sfdx cq:delta:pull -u ${PACKAGING_ORG_USERNAME} --json");
        //     def jsonResponse = readJSON text: response
        //     previousCommitId = '';
        //     if(jsonResponse.result.status == 0 && jsonResponse.result.sourceTrackingId != ''){
        //         previousCommitId = jsonResponse.result.sourceTrackingId;
        //         echo "Creating Delta from : ${previousCommitId}"
        //         project = readJSON file: 'sfdx-project.json';
        //         SOURCE_API_NAME = project.sourceApiVersion;
        //         sh "rm -rf delta"
        //         sh "mkdir -p delta"
        //         sh "sfdx sgd:source:delta -f ${previousCommitId} -o ./delta -s ./src --api-version ${SOURCE_API_NAME} -n 'package-include-files'"
        //         sh "mkdir -p delta/fullSource";
        //         sh "mv build/antscripts/bin/src delta/fullSource"
        //         sh "sfdx force:project:create -d delta --projectname . --defaultpackagedir fullSource --template empty"
        //         dir('delta'){
        //             def deltaProject = readJSON file: 'sfdx-project.json';
        //             deltaProject.sourceApiVersion = SOURCE_API_NAME;
        //             writeJSON file: 'sfdx-project.json', json: deltaProject, pretty: 4
        //             sh "sfdx force:source:convert -x package/package.xml -d metadata"
        //         }
        //     }else{
        //         error("Could not find last build history in org to continue delta deployment");
        //     }
        // }
        
        // stage('Update Package Information'){
        //     sh """
        //     cat > package.xml <<EOF
        //     `sed -n "/<?xml/,/<\\/postInstallClass>/p" src/package.xml`
        //     `sed -n "/<types>/,/<\\/Package>/p" delta/metadata/package.xml`
        //     EOF
        //     """.stripIndent();
        //     sh "mv package.xml delta/metadata/package.xml"
        //     zip zipFile: 'src.zip', archive: true, dir: 'delta/metadata', overwrite: true
        //     zip zipFile: 'destructiveChanges.zip', archive: true, dir: 'delta/destructiveChanges', overwrite: true
        // }
        
        // stage('Deploy Delta Package'){
        //     echo "Starting Delta Deployment..."
        //     sh "sfdx force:mdapi:deploy -w 200 -d delta/metadata -u ${PACKAGING_ORG_USERNAME}";

        //     try{
        //         sh "sfdx force:mdapi:deploy -w 200 -d delta/destructiveChanges -u ${PACKAGING_ORG_USERNAME}";
        //     }catch(Exception ex){
        //         echo "Failed to deploy destructive package";
        //     }
        //     sh "sfdx cq:delta:push -u ${PACKAGING_ORG_USERNAME}";
        // }
        
        // stage('Upload CQ Package'){
        //     echo "Starting Package Upload..."
        //     def cqPackageId = project.packageAliases[PACKAGE_NAME];
        //     def packageInfo = project.packageDirectories.find {element -> element.package == PACKAGE_NAME}
        //     sh "sfdx force:package1:version:create --wait 600 -u ${PACKAGING_ORG_USERNAME} --packageid ${cqPackageId} --name '${packageInfo.versionName}' --version ${packageInfo.versionNumber}";
        // }

        // bitbucketStatusNotify(buildState: 'SUCCESSFUL');
    }catch(Exception ex){
        // Since we are using try..catch block have to fail the build manually
        currentBuild.result = "FAILED";

        // Send build failed status to bitbucket
        bitbucketStatusNotify(buildState: 'FAILED');

        error(ex.getMessage());
    }
}