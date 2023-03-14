<script setup>
import * as faceapi from "face-api.js";
var video;
var canvas1;
var canvas2;
var ctxCanvas2;

const marks = reactive({
  landmarks: null,
  nose: null,
  leftEye: null,
  rightEye: null,
  jaw: null,
  leftMouth: null,
  rightMouth: null,
  leftOutline: null,
  rightOutline: null,
});
//console.log("=======", typeof marks);
const webCameraOn = () => {
  navigator.mediaDevices
    .getUserMedia({
      video: true,
      audio: false,
    })
    .then((stream) => {
      video.srcObject = stream;
      video.play();
      detect();
    })
    .catch((e) => {
      console.log(e);
    });
};

async function detect() {
  requestAnimationFrame(detect);

  // webカメラの映像から顔認識を行う
  const useTinyModel = true;
  const detection = await faceapi
    .detectSingleFace(
      video,
      new faceapi.TinyFaceDetectorOptions({
        inputSize: 160,
      })
    )
    .withFaceLandmarks(useTinyModel);

  if (!detection) {
    return;
  }

  // 認識データをリサイズ
  const resizedDetection = faceapi.resizeResults(detection, {
    width: video.width,
    height: video.height,
  });

  // ランドマークをキャンバス1に描画
  canvas1 = document.getElementById("canvas1");
  canvas1.getContext("2d").clearRect(0, 0, canvas1.width, canvas1.height);
  faceapi.draw.drawFaceLandmarks(canvas1, resizedDetection);
  // 以後使用するランドマーク座標
  marks.landmarks = resizedDetection.landmarks;
  marks.nose = marks.landmarks.getNose()[3];
  marks.leftEye = marks.landmarks.getLeftEye()[0];
  marks.rightEye = marks.landmarks.getRightEye()[3];
  marks.jaw = marks.landmarks.getJawOutline()[8];
  marks.leftMouth = marks.landmarks.getMouth()[0];
  marks.rightMouth = marks.landmarks.getMouth()[6];
  marks.leftOutline = marks.landmarks.getJawOutline()[0];
  marks.rightOutline = marks.landmarks.getJawOutline()[16];
  // canvas2
  /* コンテキスト設定 */
  ctxCanvas2.strokeStyle = "#333"; // 塗りつぶしは暗めの色
  ctxCanvas2.fillStyle = "#f00"; // 線は赤色
  ctxCanvas2.lineWidth = 5; // 線の幅は5px
  /* 円の描画 */
  ctxCanvas2.clearRect(0, 0, canvas2.width, canvas2.height);
  ctxCanvas2.beginPath(); // パスの初期化
  ctxCanvas2.arc(marks.nose._x, marks.nose._y, 30, 0, 2 * Math.PI); // (X座標, Y座標, 半径, 円の開始角度, 円の終了角度, trueで時計回り、falseで反時計回り)※角度について、1周は2π、半円はπ
  ctxCanvas2.closePath(); // パスを閉じる
  ctxCanvas2.fill(); //
  ctxCanvas2.beginPath(); // パスの初期化
  ctxCanvas2.arc(marks.leftEye._x, marks.leftEye._y, 20, 0, 2 * Math.PI); // (X座標, Y座標, 半径, 円の開始角度, 円の終了角度, trueで時計回り、falseで反時計回り)※角度について、1周は2π、半円はπ
  ctxCanvas2.closePath(); // パスを閉じる
  ctxCanvas2.fill(); //
  ctxCanvas2.beginPath(); // パスの初期化
  ctxCanvas2.arc(marks.rightEye._x, marks.rightEye._y, 20, 0, 2 * Math.PI); // (X座標, Y座標, 半径, 円の開始角度, 円の終了角度, trueで時計回り、falseで反時計回り)※角度について、1周は2π、半円はπ
  ctxCanvas2.closePath(); // パスを閉じる
  ctxCanvas2.fill(); //
}
onMounted(() => {
  video = document.getElementById("video");
  Promise.all([
    faceapi.nets.tinyFaceDetector.loadFromUri("/models/weights"),
    faceapi.nets.faceLandmark68TinyNet.loadFromUri("/models/weights"),
  ]).then(webCameraOn);
  canvas2 = document.getElementById("canvas2");
  ctxCanvas2 = canvas2.getContext("2d");
});
</script>
<template>
  <div class="grid grid-cols-2">
    <div style="position: relative" class="">
      <div>
        <video
          id="video"
          width="640"
          height="480"
          style="transform: scaleX(-1)"
        ></video>
        <canvas
          id="canvas1"
          width="640"
          height="480"
          style="position: absolute; top: 0px; left: 0px; transform: scaleX(-1)"
        ></canvas>
        <canvas
          id="canvas2"
          width="640"
          height="480"
          style="position: absolute; top: 0px; left: 0px; transform: scaleX(-1)"
        ></canvas>
      </div>
    </div>
    <div class="p-4 border">
      <p>nose:{{ marks.nose }}</p>
      <p>leftEye:{{ marks.leftEye }}</p>
      <p>rightEye:{{ marks.rightEye }}</p>
      <p>jaw:{{ marks.jaw }}</p>
      <p>leftMouth:{{ marks.leftMouth }}</p>
      <p>rightMouth:{{ marks.rightMouth }}</p>
      <p>leftOutline:{{ marks.leftOutline }}</p>
      <p>rightOutline:{{ marks.rightOutline }}</p>
    </div>
  </div>
</template>
