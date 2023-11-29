Jenkins Pipeline for Fetching and Displaying Microservice Logs 

  

Objective 

The objective is to create a Jenkins pipeline that fetches logs from multiple microservices hosted on a remote server, displays the logs for a selected microservice in the Jenkins dashboard, and archives them. 

  

Prerequisites 

1. Jenkins Server: Ensure that you have a Jenkins server installed and accessible. 

2. Remote Server: Microservice logs should be stored on a remote server accessible from the Jenkins server. 

  

Jenkinsfile Overview 

The Jenkinsfile is structured as follows: 

 1. Pipeline Setup 

  

 2. Fetch Logs Stage 

This stage fetches logs from the remote server for the selected microservice. 

The `Fetch Logs` stage in the Jenkins pipeline is responsible for retrieving logs from a remote server for the selected microservice. This stage involves connecting to the remote server, identifying the relevant logs, and copying them to the Jenkins server for further processing.  

  

Components: 

1. SSH Connection: 

The `ssh` command establishes a secure shell connection to the remote server. 

ssh -o StrictHostKeyChecking=yes -o UserKnownHostsFile=~/.ssh/known_hosts ${SERVER_USER}@${SERVER_HOST} 

  “-o StrictHostKeyChecking=yes”: Ensures strict host key checking to enhance security. 

  “-o UserKnownHostsFile=~/.ssh/known_hosts”: Specifies the known hosts file to check for server authenticity. 

2. Find and Group Logs by Directory: 

The `find` command searches for all files (`-type f`) with a `.log` extension in the specified `${LOGS_BASE_PATH}`. 

The `-exec dirname {} \\;` part extracts the directory name for each found log file. 

The result is sorted and unique directory names are written to `microservice_directories.txt`. 

find ${LOGS_BASE_PATH} -type f -name "*.log" -exec dirname {} \; | sort -u > microservice_directories.txt 

3. Read and Copy Logs: 

The `while` loop reads each line (microservice directory) from `microservice_directories.txt`. 

The `scp` command copies all `*.log` files from the specified directory on the remote server to the current workspace on the Jenkins server. 

 while IFS= read -r dir; do scp -o StrictHostKeyChecking=yes -o UserKnownHostsFile=~/.ssh/known_hosts -r ${SERVER_USER}@${SERVER_HOST}:${dir}/*.log .; done < microservice_directories.txt 

 `IFS= read -r dir`: Reads each line without interpreting backslashes and whitespace. 

Workflow: 

1. Establish SSH Connection: 

Connect securely to the remote server using SSH. 

2. Find Relevant Logs: 

Use the `find` command to locate all `.log` files in the specified microservice directory. 

3. Group Logs by Directory: 

Extract the directory names for each log file and sort them. 

4. Copy Logs to Jenkins Server: 

Use the `scp` command to copy all `*.log` files from each microservice directory to the Jenkins server. 

Notes: 

`${SERVER_USER}`, `${SERVER_HOST}`, and `${LOGS_BASE_PATH}` are environment variables set in the Jenkins pipeline. 

The fetched logs will be used in subsequent stages, such as displaying them in HTML reports and archiving them. 

This `Fetch Logs` stage ensures a systematic and secure retrieval of microservice logs, setting the foundation for further processing and analysis in the Jenkins pipeline. 

  

3. Publish HTML Reports Stage 

 
Certainly! The Publish HTML Reports stage in the Jenkins pipeline is responsible for displaying HTML reports based on the fetched logs from the remote server. This stage involves checking if there are log files available, and if so, publishing them as HTML reports in the Jenkins job dashboard. Let's break down the key components of this stage: 

  

Components: 

1.Check for Log Files: 

The ls *.log command is executed to list all .log files in the current workspace directory. 

The returnStatus: true option returns a non-zero status if no files are found. 

The result is stored in the logFilesExist variable. 

2.Publish HTML Reports: 

The presence of log files is checked using the logFilesExist variable. 

If log files exist, the publishHTML step is executed. 

 

allowMissing: false: Fails the build if the specified HTML report files are missing. 

alwaysLinkToLastBuild: true: Ensures that the HTML report is linked to the last build, even if it is empty. 

keepAll: true: Retains HTML reports for all builds. 

reportDir: '.': Specifies the directory containing the HTML report files (current workspace). 

reportFiles: '*.log': Defines the pattern of HTML report files to include (all .log files). 

reportName: 'Microservice Logs': Sets the name of the HTML report displayed in the Jenkins job dashboard. 

3.Display Message for No Logs: 

If no log files are found, an informational message is echoed. 

 

Workflow: 

Check for Log Files: 

List all .log files in the current workspace. 

 

Publish HTML Reports: 

If log files are found, use publishHTML to display them as an HTML report in the Jenkins dashboard. 

Display Message for No Logs: 

If no log files are found, echo an informational message. 

Notes: 

The publishHTML step is conditional on the existence of log files to prevent unnecessary steps if no logs are fetched. 

The HTML report is linked to the last build, allowing easy navigation to the logs for each build. 

A message is displayed if no log files are found, providing transparency to users. 

This Publish HTML Reports stage enhances the Jenkins job dashboard by displaying microservice logs as HTML reports, facilitating visibility and analysis of log data for each build. 

 
4. Archive Logs Stage  

This stage archives the fetched logs. 

 The Archive Logs stage in the Jenkins pipeline is responsible for archiving log files to preserve them for later access. This stage typically comes after the Publish HTML Reports stage and ensures that log files are stored as artifacts, allowing users to download or access them later. 

 

 

Archive Artifacts: 

The archiveArtifacts step is used to archive specific files or directories as artifacts of the Jenkins build. 

                                 archiveArtifacts artifacts: '*.log', allowEmptyArchive: true 
 

artifacts: '*.log': Specifies the pattern of files to be archived (in this case, all .log files). 

allowEmptyArchive: true: Allows archiving even if no matching artifacts are found. This is useful to prevent build failures when no logs are available. 

 

 

 

Workflow: 

Archive Artifacts: 

The archiveArtifacts step archives all .log files in the current workspace. 

 

The pattern */**/*.log is a file glob pattern used to match files with a specific extension (.log in this case) in a directory and its subdirectories 

*/: Matches any directory (or subdirectory) at the current level. 

**/: Recursively matches directories at any level. 

*.log: Matches any file with a .log extension. 

When combined, */**/*.log effectively matches all .log files in the current directory and its subdirectories. 

 

Notes: 

The archiveArtifacts step is typically used after stages that generate important artifacts, such as log files or build outputs. 

Archiving artifacts is useful for preserving build-related information and ensuring that users can access or download them later. 

The allowEmptyArchive option is set to true to prevent build failures when no log files are available. This allows for a graceful handling of scenarios where logs may not be generated. 

Example Use Case: 

Suppose you have a Jenkins job that fetches and displays logs from a remote server, and you want to archive these logs for future reference or analysis. The Archive Logs stage ensures that the logs are stored as artifacts, and users can access them through the Jenkins web interface or download them for further investigation. 

This stage is particularly valuable for preserving artifacts across different builds, facilitating traceability, and ensuring that historical build information remains accessible even after subsequent builds have taken place. 

 

5. Workspace Cleanup 

The `post` section ensures the workspace is cleaned after the build. 

 

Usage 

1. Triggering the Pipeline: Manually trigger the pipeline with a specified microservice. 

2. Viewing Logs: View logs for the selected microservice in the Jenkins dashboard. 

3. Archiving Logs: Archived logs can be downloaded for future reference. 

 

This Jenkins pipeline provides a structured approach to fetch, display, and archive microservice logs. 