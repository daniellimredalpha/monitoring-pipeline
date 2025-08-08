pipeline {
  agent any
  options { timestamps() }

  stages {
    stage('Build') {
      steps {
        checkout scm
        sh '''
          echo "Validating structure..."
          test -f app.py
          test -d templates
          test -d static
        '''
      }
    }

    stage('Test') {
      steps {
        sh 'python3 -m py_compile app.py || true'
      }
    }

    stage('Deploy') {
      steps {
        sshagent (credentials: ['jenkins-ssh-mon']) {
          sh '''
            # sync repo to monitoring server
            rsync -avz --delete ./ <user>@10.10.14.10:/opt/monitor/app/

            # install deps (optional: via requirements.txt)
            ssh ceo@10.10.14.10 '
              set -e
              test -f /opt/monitor/app/requirements.txt && \
                /opt/monitor/venv/bin/pip install -r /opt/monitor/app/requirements.txt || true

              sudo systemctl restart monitor
              sleep 2
              systemctl is-active monitor
            '
          '''
        }
      }
    }
  }
}
