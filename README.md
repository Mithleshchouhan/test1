<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cast Video Controls</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin-top: 50px;
        }
        video {
            width: 600px;
            margin-bottom: 10px;
        }
        button {
            margin: 5px;
            padding: 10px 20px;
            font-size: 16px;
        }
        input[type="number"] {
            width: 80px;
            padding: 5px;
        }
        #castButton {
            background-color: #4285f4;
            color: white;
            padding: 10px 20px;
            font-size: 16px;
            border: none;
            cursor: pointer;
            border-radius: 5px;
        }
    </style>
</head>
<body>

    <h1>Cast Video with Forward, Reverse, and Seek Controls</h1>

    <!-- Cast button -->
    <button id="castButton">Cast to TV</button>

    <!-- Video element -->
    <video id="myVideo" controls>
        <!-- Replace 'video.mp4' with your actual video link -->
        <source src="https://storage.googleapis.com/gtv-videos-bucket/sample/TearsOfSteel.mp4" type="video/mp4">
        Your browser does not support HTML video.
    </video>
    <br>

    <button onclick="rewind()">Rewind 10s</button>
    <button onclick="forward()">Forward 10s</button>
    <br>

    <input type="number" id="seekTime" placeholder="Enter seconds">
    <button onclick="seek()">Seek to Time</button>

    <!-- Google Cast SDK Script -->
    <script type="text/javascript" src="https://www.gstatic.com/cv/js/sender/v1/cast_sender.js?loadCastFramework=1"></script>

    <script>
        var video = document.getElementById("myVideo");
        var session = null;

        function initializeCastApi() {
            cast.framework.CastContext.getInstance().setOptions({
                receiverApplicationId: chrome.cast.media.DEFAULT_MEDIA_RECEIVER_APP_ID,  // Using Default Receiver
                autoJoinPolicy: chrome.cast.AutoJoinPolicy.ORIGIN_SCOPED
            });
        }

        // Rewind 10 seconds
        function rewind() {
            if (session) {
                var currentTime = session.getMediaSession().currentTime;
                session.getMediaSession().seek({ currentTime: currentTime - 10 });
            } else {
                video.currentTime -= 10;
            }
        }

        // Forward 10 seconds
        function forward() {
            if (session) {
                var currentTime = session.getMediaSession().currentTime;
                session.getMediaSession().seek({ currentTime: currentTime + 10 });
            } else {
                video.currentTime += 10;
            }
        }

        // Seek to specific time
        function seek() {
            var seekTime = document.getElementById("seekTime").value;
            if (session) {
                session.getMediaSession().seek({ currentTime: seekTime });
            } else {
                video.currentTime = seekTime;
            }
        }

        // Initialize Google Cast
        window['__onGCastApiAvailable'] = function(isAvailable) {
            if (isAvailable) {
                initializeCastApi();
            }
        };

        // Cast Button Click Event
        document.getElementById("castButton").addEventListener("click", function() {
            var castSession = cast.framework.CastContext.getInstance().getCurrentSession();
            if (!castSession) {
                cast.framework.CastContext.getInstance().requestSession().then(() => {
                    session = cast.framework.CastContext.getInstance().getCurrentSession();
                    castVideo();
                });
            } else {
                session = castSession;
                castVideo();
            }
        });

        function castVideo() {
            var mediaInfo = new chrome.cast.media.MediaInfo(video.src, 'video/mp4');
            var request = new chrome.cast.media.LoadRequest(mediaInfo);
            session = cast.framework.CastContext.getInstance().getCurrentSession();
            session.loadMedia(request).then(function() {
                console.log("Media loaded successfully.");
            }).catch(function(error) {
                console.error("Error loading media: " + error);
            });
        }
    </script>

</body>
</html>
