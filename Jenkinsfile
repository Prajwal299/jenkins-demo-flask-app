stage('Transfer Files to EC2') {
    steps {
        withCredentials([sshUserPrivateKey(
            credentialsId: 'docker-jenkins-app', // your Jenkins EC2 SSH key
            keyFileVariable: 'SSH_KEY',
            usernameVariable: 'SSH_USER'
        )]) {
            script {
                bat """
                    powershell -Command "
                        \$acl = Get-Acl '%SSH_KEY%';
                        \$acl.SetAccessRuleProtection(\$true, \$false);
                        \$acl.Access | Where-Object { \$_.IdentityReference -ne 'NT AUTHORITY\\SYSTEM' -and \$_.IdentityReference -ne \$env:USERNAME } | ForEach-Object { \$acl.RemoveAccessRule(\$_) };
                        Set-Acl '%SSH_KEY%' \$acl
                    "
                """

                bat """
                    powershell -Command "scp -i %SSH_KEY% -o StrictHostKeyChecking=no -r * ${EC2_HOST}:${EC2_APP_DIR}"
                """
            }
        }
    }
}
