# Emotion Enabled Videoconferencing 

## Install
0. Build out the charts/ directory from the Chart.lock file (before that you might need to have the repositories in Chart.yaml added to helm by `helm add repo`):

        helm dependency build ./

1. Install the chart:

        helm install fh ./

    - or deploy with a Redis Gears script that assumes images are already portraits and therefore no face detection model needs to run

            helm install --set redisAIProvider.faceRecognitionEnabled=false 