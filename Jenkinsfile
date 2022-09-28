pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
                sh 'echo deploy aaa'
                //sh 'chmod +x ./src/hl'
                //sh 'scp  -r -i /home/.ssh/id_rsa ./src/ jesus@192.168.6.41:/srv/scripts/'
            }
        }
        stage('Notificar') {
            steps {
                sh 'echo va pa tele'
                sh 'curl "https://api.telegram.org/bot1356318701:AAHNg-CnpKEl1sbvI_fOH3U64fRl0A4iiuM/sendMessage?chat_id=-1001382570574&text=Se realizo el deploy del script hl,mediante Jenkins"'
            }
        }
    }
}
