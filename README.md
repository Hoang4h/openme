<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Heart Animation</title>

    <style>
        body {
            background-color: #000;
            margin: 0;
            overflow: hidden;
        }

        #heart {
            position: absolute;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
        }
    </style>
</head>
<body>

    <canvas id="heart"></canvas>

    <script>
        document.addEventListener('DOMContentLoaded', function () {
            var canvas = document.getElementById('heart');
            var ctx = canvas.getContext('2d');

            function resizeCanvas() {
                canvas.width = window.innerWidth;
                canvas.height = window.innerHeight;
            }

            resizeCanvas();
            window.addEventListener('resize', resizeCanvas);

            var heartPosition = function (rad) {
                return [
                    Math.pow(Math.sin(rad), 3),
                    -(15 * Math.cos(rad) - 5 * Math.cos(2 * rad) - 2 * Math.cos(3 * rad) - Math.cos(4 * rad))
                ];
            };

            var scaleAndTranslate = function (pos, sx, sy, dx, dy) {
                return [dx + pos[0] * sx, dy + pos[1] * sy];
            };

            var pointsOrigin = [];
            var dr = 0.1;
            for (var i = 0; i < Math.PI * 2; i += dr) {
                pointsOrigin.push(scaleAndTranslate(heartPosition(i), 210, 13, 0, 0));
            }

            var targetPoints = [];
            function pulse(kx, ky) {
                for (var i = 0; i < pointsOrigin.length; i++) {
                    targetPoints[i] = [
                        kx * pointsOrigin[i][0] + canvas.width / 2,
                        ky * pointsOrigin[i][1] + canvas.height / 2
                    ];
                }
            }

            var e = [];
            for (var i = 0; i < pointsOrigin.length; i++) {
                var x = Math.random() * canvas.width;
                var y = Math.random() * canvas.height;
                e[i] = {
                    vx: 0,
                    vy: 0,
                    R: 2,
                    speed: Math.random() + 5,
                    q: ~~(Math.random() * pointsOrigin.length),
                    D: 2 * (i % 2) - 1,
                    force: 0.2 * Math.random() + 0.7,
                    f: "hsla(0," + ~~(40 * Math.random() + 60) + "%," + ~~(60 * Math.random() + 20) + "%,.3)",
                    trace: []
                };
                for (var k = 0; k < 50; k++) e[i].trace[k] = { x: x, y: y };
            }

            var time = 0;
            function loop() {
                var n = -Math.cos(time);
                pulse((1 + n) * 0.5, (1 + n) * 0.5);
                time += 0.01;

                ctx.fillStyle = "rgba(0,0,0,0.1)";
                ctx.fillRect(0, 0, canvas.width, canvas.height);

                for (var i = e.length; i--;) {
                    var u = e[i];
                    var q = targetPoints[u.q];
                    var dx = u.trace[0].x - q[0];
                    var dy = u.trace[0].y - q[1];
                    var length = Math.sqrt(dx * dx + dy * dy);
                    if (length < 10) {
                        if (Math.random() > 0.95) {
                            u.q = ~~(Math.random() * pointsOrigin.length);
                        } else {
                            if (Math.random() > 0.99) u.D *= -1;
                            u.q = (u.q + u.D + pointsOrigin.length) % pointsOrigin.length;
                        }
                    }
                    u.vx += (-dx / length) * u.speed;
                    u.vy += (-dy / length) * u.speed;
                    u.trace[0].x += u.vx;
                    u.trace[0].y += u.vy;
                    u.vx *= u.force;
                    u.vy *= u.force;

                    for (var k = 0; k < u.trace.length - 1; k++) {
                        u.trace[k + 1].x -= 0.4 * (u.trace[k + 1].x - u.trace[k].x);
                        u.trace[k + 1].y -= 0.4 * (u.trace[k + 1].y - u.trace[k].y);
                    }

                    ctx.fillStyle = u.f;
                    for (var k = 0; k < u.trace.length; k++) {
                        ctx.fillRect(u.trace[k].x, u.trace[k].y, 1, 1);
                    }
                }

                requestAnimationFrame(loop);
            }

            loop();
        });
    </script>

</body>
</html>
