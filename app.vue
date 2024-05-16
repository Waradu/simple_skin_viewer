<template>
  <canvas id="viewer" ref="canva"></canvas>
</template>

<script lang="ts" setup>
import { SkinViewer } from "skinview3d";

const skinPath = ref("/skin.png");

onMounted(() => {
  const skinViewer = new SkinViewer({
    canvas: document.getElementById("viewer") as HTMLCanvasElement,
    width: window.innerWidth,
    height: window.innerHeight,
    zoom: 0.5,
    skin: skinPath.value,
    panorama: "/panorama.jpg",
  });

  const resizeHandler = () => {
    skinViewer.width = window.innerWidth;
    skinViewer.height = window.innerHeight;
  };

  window.addEventListener("resize", resizeHandler);

  const fetchSkin = async () => {
    try {
      const response = await fetch(skinPath.value, { cache: "no-store" });
      if (response.ok) {
        const blob = await response.blob();
        const newSkinURL = URL.createObjectURL(blob);

        const model: "default" | "slim" = await determineSkinModel(blob);

        skinViewer.loadSkin(newSkinURL, { model });
      } else {
        console.error("Failed to fetch skin file:", response.status);
      }
    } catch (error) {
      console.error("Error fetching skin file:", error);
    }
  };

  const determineSkinModel = (blob: Blob): Promise<"default" | "slim"> => {
    return new Promise((resolve, reject) => {
      const img = new Image();
      img.onload = () => {
        const canvas = document.createElement("canvas");
        canvas.width = img.width;
        canvas.height = img.height;
        const ctx = canvas.getContext("2d");
        if (ctx) {
          ctx.drawImage(img, 0, 0);
          const imageData = ctx.getImageData(0, 0, img.width, img.height);
          const pixelIndex =
            (img.width * (img.height - 2) + (img.width - 5)) * 4 + 3;
          const alphaValue = imageData.data[pixelIndex];
          resolve(alphaValue === 0 ? "slim" : "default");
        } else {
          reject(new Error("Failed to get canvas context"));
        }
      };
      img.onerror = reject;
      img.src = URL.createObjectURL(blob);
    });
  };

  fetchSkin()
  setInterval(fetchSkin, 1000);
});
</script>

<style lang="scss">
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html,
body,
#__nuxt {
  width: 100%;
  height: 100%;
  background-color: #0c0c0c;
  overflow: hidden;
}
</style>
