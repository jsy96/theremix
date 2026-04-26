<script>
  import { videoReady, mediapipeHandsLoaded, modelLoaded,handPoseRes,isSafari,allowHolistic,PoseFTUE } from "../stores.js";
  import { smooth, getDistance } from "../helpers.js";

  function onResults(results) {

    handPoseRes.set(results)


     if (!$modelLoaded) {
        modelLoaded.set(true);
    }
}

  const holistic = new Holistic({locateFile: (file) => {
  return `https://cdn.jsdelivr.net/npm/@mediapipe/holistic@0.1/${file}`;
}});
holistic.setOptions({
  upperBodyOnly: true,
  smoothLandmarks: true,
  minDetectionConfidence: 0.75,
  minTrackingConfidence: 0.5
});
holistic.onResults(onResults);

  let running = false;

  async function estimatePoseOnImage() {
    if ($videoReady && $allowHolistic) {
      running = true;
      if($videoReady.width!==$videoReady.videoWidth || $videoReady.height!==$videoReady.videoHeight){
        $videoReady.width = $videoReady.videoWidth
        $videoReady.height = $videoReady.videoHeight
      }
      try {
        await holistic.send({image: $videoReady});
      } catch(e) {
        console.warn('Holistic detection error:', e);
      }
      requestAnimationFrame(estimatePoseOnImage);
    } else {
      running = false;
      handPoseRes.set(null)
    }
  }

  $: {
    if ($videoReady && holistic && $allowHolistic && !isSafari && !$PoseFTUE && !running) {
      estimatePoseOnImage();
    }
  }
</script>
