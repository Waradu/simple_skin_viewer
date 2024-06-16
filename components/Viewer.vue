<template>
  <canvas id="viewer" ref="canva"></canvas>
</template>

<script lang="ts" setup>
import { IdleAnimation, SkinViewer, WalkingAnimation } from "skinview3d";
import type { BodyPart } from "~/types/types";

const props = defineProps({
  model: {
    type: String,
    default: "auto-detect",
  },
  animation: {
    type: String,
    default: "none",
  },
  layers: {
    type: Array,
    default: () => [
      {
        name: "Head",
        value: "head",
        inner: true,
        outer: true,
      },
      {
        name: "Body",
        value: "body",
        inner: true,
        outer: true,
      },
      {
        name: "Right Arm",
        value: "rightArm",
        inner: true,
        outer: true,
      },
      {
        name: "Left Arm",
        value: "leftArm",
        inner: true,
        outer: true,
      },
      {
        name: "Right Leg",
        value: "rightLeg",
        inner: true,
        outer: true,
      },
      {
        name: "Left Leg",
        value: "leftLeg",
        inner: true,
        outer: true,
      },
    ],
  },
  skin: {
    type: String,
    default: "/skin.png",
  },
});

const model = ref(props.model);
const animation = ref(props.animation);
const skin = ref(props.skin);

let skinViewer: SkinViewer | null = null;

onMounted(() => {
  skinViewer = new SkinViewer({
    canvas: document.getElementById("viewer") as HTMLCanvasElement,
    width: window.innerWidth,
    height: window.innerHeight,
    zoom: 0.8,
    skin: skin.value,
    panorama: "/panoramas/panorama.jpg",
    model: model.value as "default" | "slim" | "auto-detect",
  });

  const resizeHandler = () => {
    skinViewer!.width = window.innerWidth;
    skinViewer!.height = window.innerHeight;
  };

  window.addEventListener("resize", resizeHandler);

  setAnimation(animation.value as "none" | "idle" | "walk");
});

watch(
  () => props.model,
  (newModel: any) => {
    if (skinViewer) {
      skinViewer.loadSkin(skin.value, {
        model: newModel,
      });
    }
  }
);

watch(
  () => props.skin,
  (newSkin: any) => {
    if (skinViewer) {
      skinViewer.loadSkin(newSkin);
      if (newSkin) {
        extractAndSetFavicon(newSkin);
      }
    }
  }
);

watch(
  () => props.animation,
  (newAnimation: any) => {
    setAnimation(newAnimation);
  }
);

watch(
  () => props.layers,
  (newLayers: any) => {
    newLayers.forEach((layer: any) => {
      if (skinViewer) {
        const part = layer.value as BodyPart;

        skinViewer.playerObject.skin[part].innerLayer.visible = layer.inner;
        skinViewer.playerObject.skin[part].outerLayer.visible = layer.outer;
      }
    });
  },
  { deep: true }
);

function setAnimation(animation: string) {
  if (!skinViewer) return;

  switch (animation) {
    case "idle":
      skinViewer.animation = new IdleAnimation();
      break;
    case "walk":
      skinViewer.animation = new WalkingAnimation();
      break;
    case "none":
    default:
      skinViewer.animation = null;
      break;
  }
}

function extractAndSetFavicon(skin: string) {
  const canvas = document.createElement("canvas");
  const context = canvas.getContext("2d");
  const img = new Image();
  img.src = skin;

  if (!context) return;

  img.onload = () => {
    canvas.width = 8;
    canvas.height = 8;
    context.drawImage(img, -8, -8);

    const faviconCanvas = document.createElement("canvas");
    const faviconContext = faviconCanvas.getContext("2d");
    faviconCanvas.width = 32;
    faviconCanvas.height = 32;

    if (!faviconContext) return;

    faviconContext.imageSmoothingEnabled = false;
    faviconContext.drawImage(canvas, 0, 0, 8, 8, 0, 0, 32, 32);

    const faviconUrl = faviconCanvas.toDataURL("image/png");

    useHead({
      link: [
        {
          rel: "icon",
          href: faviconUrl,
        },
      ],
    });
  };
}

function screenShot() {
  if (!skinViewer) return;

  skinViewer.width = 1920;
  skinViewer.height = 1080;
  skinViewer.zoom = 0.8;

  skinViewer.render();

  const image = skinViewer.canvas.toDataURL();

  const link = document.createElement("a");
  link.href = image;
  link.download = "render.png";
  link.click();

  skinViewer!.width = window.innerWidth;
  skinViewer!.height = window.innerHeight;
}

function reset(skin: string) {
  if (!skinViewer) return;

  skinViewer.dispose();

  extractAndSetFavicon(skin);

  skinViewer = new SkinViewer({
    canvas: document.getElementById("viewer") as HTMLCanvasElement,
    width: window.innerWidth,
    height: window.innerHeight,
    zoom: 0.8,
    skin: skin,
    panorama: "/panoramas/panorama.jpg",
    model: model.value as "default" | "slim" | "auto-detect",
  });
}

defineExpose({ screenShot, reset });
</script>

<style lang="scss">
#viewer {
  cursor: move;
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
}
</style>
