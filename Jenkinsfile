// SYSTEM SETTINGS
WAITING_TIME=240 // in seconds.

INFO_TAG='[INFO]'
ERROR_TAG='[ERROR]'
DEBUG_TAG='[DEBUG]'
WARN_TAG='[WARN]'
SUCCESS_TAG='[SUCCESS]'
FAILURE_TAG='[FAILURE]'
ABORT_TAG='[ABORT]'

ENABLE_SLACK_NOTIFICATION = true

enum Colors
{
  BLUE('#009fdb', '\u001B[34m'),
  GREEN('#60c36e', '\u001B[32m'),
  RED('#db3c00', '\u001B[31m'),
  YELLOW('#ffd700', '\u001B[33m'),
  PURPLE('#9e3b7e', '\u001B[35m');

  public String hexa_code
  public String xterm_code

  public Colors(String hexa_code, xterm_code)
  {
    this.hexa_code = hexa_code
    this.xterm_code = xterm_code
  }
}

pipeline
{
  agent
  {
    node
    {
      // Execute this job only on mkdocs ready node.
      label 'esgf-docker-slave-2'

      // Don't let Jenkins generate the workspace name: it is too long and
      // crashes the ESGF config generation stage.
      // env.WORKSPACE is unknown at this step, so we cannot do better than
      // provide an absolute path.
      customWorkspace "/home/jenkins/slave_home/workspace/${env.JOB_NAME}"
    }
  }

  options
  {
    ansiColor('xterm') // Interprets color. Needs AnsiColor plugin.
    timeout(time: 5, unit: 'MINUTES')
    timestamps() // Add timestamp. Needs TimeStamp plugin.
  }

  environment
  {
    /*** SYSTEM PATHS ***/
    DOCS_DIR_PATH="${env.WORKSPACE}/mkdocs"
    MKDOCS_SITE_DIR_PATH="${DOCS_DIR_PATH}/site"

    /*** GITHUB ***/
    DOCS_REPO_URL='https://github.com/Prodiguer/docs.git'
    BRANCH_NAME='master'

    /*** SLACK ***/
    SLACK_CHANNEL='#docs'
    SLACK_CREDENTIAL_ID='slack_prodiguer_docs'
    SLACK_MSG_PREFIX="ESPRI-MOD Docs <${env.BUILD_URL}|${BRANCH_NAME}#${env.BUILD_ID}>:"

    /*** CONDA ***/
    CONDA_ENV_BIN_DIR_PATH='/home/jenkins/miniconda3/envs/mkdocs/bin'

    /*** CONSOL OUTPUT ***/
    CONSOLE_MSG_PREFIX="commit on ${BRANCH_NAME}"
  }

  stages
  {
    stage('checkout')
    {
      steps
      {
        start_block('checkout')

        info('checkout docs repository')
        git(url: DOCS_REPO_URL, branch: BRANCH_NAME)

        end_block('checkout')
      }
    }

    stage('build')
    {
      steps
      {
        start_block('build')
        dir(DOCS_DIR_PATH)
        {
          info('build mkdocs')    
          sh'${CONDA_ENV_BIN_DIR_PATH}/python ${CONDA_ENV_BIN_DIR_PATH}/mkdocs build'
        }
        end_block('build')
      }
    }

    stage('deploy')
    {
      steps
      {
        start_block('deploy')
        dir(DOCS_DIR_PATH)
        {
          info('zip generated site')    
          sh'zip -r site.zip site'
          archiveArtifacts(artifacts: 'site.zip', onlyIfSuccessful: true)
          sh'rm -fr /var/html/mkdocs/site'
          sh'cp -rp site /var/html/mkdocs'
        }
        end_block('deploy')
      }
    }
  }

  post
  {
    failure
    {
      info('delete the workspace on failure')
      deleteDir() // safe precaution
      failure("${env.CONSOLE_MSG_PREFIX} on previous error(s)")
    }

    success
    {
      info('delete the workspace on success')
      deleteDir() // safe precaution
      success("${env.CONSOLE_MSG_PREFIX}")
    }

    aborted
    {
      info('delete the workspace on abortion')
      deleteDir() // safe precaution
      abort("${env.CONSOLE_MSG_PREFIX}")
    }
  }
}


def start_block(block_name)
{
  debug("BEGIN BLOCK ${block_name} ------------------------------------------------------")
}

def end_block(block_name)
{
  debug("END BLOCK ${block_name} --------------------------------------------------------")
}

def begin(msg)
{
  console_output(msg, INFO_TAG, Colors.BLUE)
  slack_send(msg, Colors.BLUE)
}

def success(msg)
{
  console_output(msg, SUCCESS_TAG, Colors.GREEN)
  slack_send('*SUCCESS*', Colors.GREEN)
  currentBuild.result = 'SUCCESS'
}

def failure(msg)
{
  console_output(msg, FAILURE_TAG, Colors.RED)
  slack_send('*FAILURE*', Colors.RED)
  currentBuild.result = 'FAILURE'
}

def abort(msg)
{
  console_output(msg, ABORT_TAG, Colors.RED)
  slack_send('*Aborted*', Colors.RED)
  currentBuild.result = 'ABORTED'
}

def info(msg)
{
  console_output(msg, INFO_TAG, Colors.BLUE)
}

def error(msg)
{
  console_output(msg, ERROR_TAG, Colors.RED)
}

def warn(msg)
{
  console_output(msg, WARN_TAG, Colors.YELLOW)
}

def debug(msg)
{
  console_output(msg, DEBUG_TAG, Colors.PURPLE)
}

def slack_send(msg, color)
{
  msg = "${env.SLACK_MSG_PREFIX} ${msg}"

  if(ENABLE_SLACK_NOTIFICATION)
  {
    withCredentials([usernamePassword(usernameVariable: 'slack_url', passwordVariable: 'slack_token', credentialsId: "${SLACK_CREDENTIAL_ID}")])
    {
      slackSend(message: msg, color: color.hexa_code, baseUrl: slack_url, channel: SLACK_CHANNEL, token: slack_token, botUser: true)
    }
  }
  else
  {
    echo("slack notifications are disable. Message was ${msg}")
  }
}

def console_output(msg, tag, color)
{
  echo(String.format("%s%s %s%s", color.xterm_code, tag, msg, '\u001B[0m'))
}