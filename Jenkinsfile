pipeline {
    // تشغيل الـ Pipeline على أي Agent متاح (في حالتك: Built-In Node)
    agent any 

    // تعريف متغيرات البيئة الرئيسية المستخدمة في الـ Pipeline
    environment {
        // اسم الصورة على Docker Hub (تم تحديثه ليتطابق مع اسم حسابك)
        DOCKER_IMAGE_NAME = "alialhoumsi/docker-ci-demo" 
        // اسم مفتاح الـ Credentials الخاص بـ Docker Hub (كما استخدمته في الإعدادات)
        DOCKER_CREDENTIAL_ID = 'dockerhub-creds' 
        // سيتم تعريفه لاحقاً: env.IMAGE_TAG
    }

    stages {

        stage('01. Checkout Code') {
            steps {
                // سحب الكود من SCM (Git)
                checkout scm 
                echo '📥 Code checked out successfully.'
            }
        }

        // --- مرحلة بناء صورة Docker ---
        stage('02. Build Docker Image') {
            steps {
                script {
                    // 1. الحصول على الـ commit hash القصير واستخدامه كـ Tag
                    def shortSha = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    env.IMAGE_TAG = "${DOCKER_IMAGE_NAME}:${shortSha}" // مثال: alialhoumsi/docker-ci-demo:1d7647e

                    echo "🚧 Building image: ${env.IMAGE_TAG}"

                    // 2. أمر بناء الصورة مع استخدام multi-line string لتجنب مشاكل المسافات
                    sh """
                        docker build -t ${env.IMAGE_TAG} \
                                     -t ${DOCKER_IMAGE_NAME}:latest \
                                     .
                    """

                    echo "📦 Docker images built successfully."
                }
            }
        }

        // --- مرحلة تسجيل الدخول إلى Docker Hub باستخدام Credentials ---
        stage('03. Login to Docker Hub') {
            steps {
                // استخدام معرّف الـ Credentials المخزّن في Jenkins
                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIAL_ID}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "🔑 Logging in with user: $DOCKER_USER"
                        // تسجيل الدخول باستخدام كلمة المرور عبر STDIN لتفادي ظهورها في الـ logs
                        echo "$DOCKER_PASS" | docker login \
                            --username "$DOCKER_USER" \
                            --password-stdin
                    '''
                }
            }
        }

        // --- مرحلة دفع الصورة (Push) ---
        stage('04. Push Image to Docker Hub') {
            steps {
                echo "📤 Pushing images to Docker Hub..."
                
                sh """
                    // دفع الصورة بالـ Tag الخاص بالـ commit
                    docker push ${env.IMAGE_TAG}
                    // دفع الصورة بـ Tag الـ 'latest'
                    docker push ${DOCKER_IMAGE_NAME}:latest
                """

                echo "🎉 Successfully pushed: ${env.IMAGE_TAG} and latest"
            }
        }
    }

    // --- الإجراءات التي تتم بعد انتهاء الـ Pipeline (Post-build Actions) ---
    post {
        // تنفذ دائماً (سواء نجح الـ Pipeline أو فشل)
        always {
            echo "🧹 Cleaning up and logging out..."
            // تسجيل الخروج من Docker Hub (حتى لو فشل أي شيء، لضمان الأمان)
            sh "docker logout || true" 
            // حذف الصور والحاويات غير المستخدمة لتوفير مساحة القرص (مهم جداً لحل مشكلتك)
            sh "docker system prune -a -f || true" 
        }
        success {
            echo "✔ Pipeline completed successfully! Image is ready on Docker Hub."
        }
        failure {
            echo "❌ Pipeline failed! Check the Console Output for Docker errors."
        }
    }
}
