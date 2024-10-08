pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('[ZAP] Baseline passive-scan') {
            steps {
                sh '''
            docker run --name juice-shop -d --rm \
                -p 3000:3000 \
                bkimminich/juice-shop
            sleep 5
        '''
                sh '''
            docker run --name zap  \
                --add-host=host.docker.internal:host-gateway \
                -v /tmp/zap:/zap/wrk/:rw \
                -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive_scan.yaml" \
                || true
        '''
            }
            post {
                always {
                    sh '''
                docker stop juice-shop
                docker cp zap:/zap/wrk/reports/zap_xml_report.xml /tmp/zap_xml_report.xml
                docker rm zap

            '''
                    defectDojoPublisher(artifact: '/tmp/zap_xml_report.xml',
                    productName: 'Juice Shop',
                    scanType: 'ZAP Scan',
                    engagementName: 'mwwski@gmail.com')
                }
            }
        }
    }
}
