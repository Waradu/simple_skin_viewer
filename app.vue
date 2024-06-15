<template>
  <div class="context" v-if="!max">
    <h2>Waradu's Skin Editor</h2>
    <p>
      Click 'Change Skin' and select the skin file. Then, edit the local skin
      file with your preferred image or skin editor, and watch the website
      auto-update your skin in real-time.
    </p>
  </div>
  <Viewer
    :model="model.value"
    :animation="animation.value"
    :layers="layers"
    :skin="skin"
    ref="viewerRef"
  />
  <div class="overlay">
    <div class="left" :class="{ hidden: max }">
      <div class="wrdu-floating skin">
        <div class="image wrdu-element active" v-if="skin">
          <img :src="skin" alt="skin" />
        </div>
        <div class="upload">
          <button id="upload">Change Skin</button>
        </div>
      </div>
      <div class="hover-area"></div>
    </div>
    <div class="center" :class="{ hidden: max }">
      <div class="actions wrdu-floating">
        <div class="action wrdu-element screenshot" @click="screenShot">
          <Iconsax name="Camera" thickness="2" />
          <div class="hover">
            <h3>Take Screenshot</h3>
            <p>Take and save a screenshot of the current canvas</p>
          </div>
        </div>
        <div class="action wrdu-element maximize" @click="max = !max">
          <Iconsax name="Maximize3" thickness="2" v-if="!max" />
          <Iconsax
            name="Add"
            thickness="2"
            v-if="max"
            :style="{ scale: '1.4', rotate: '45deg' }"
          />
          <div class="hover" v-if="!max">
            <h3>Hide UI</h3>
            <p>Move mouse to the side to show part of the UI</p>
          </div>
          <div class="hover" v-if="max">
            <h3>Show UI</h3>
            <p>Shows the UI again</p>
          </div>
        </div>
        <div class="action wrdu-element reset" @click="reset">
          <Iconsax name="ArrowRotateLeft" thickness="2" />
          <div class="hover">
            <h3>Reset</h3>
            <p>Reset Everything back to normal</p>
          </div>
        </div>
      </div>
      <div class="hover-area"></div>
    </div>
    <div class="right" :class="{ hidden: max }">
      <div class="wrdu-window wrdu-vertical">
        <div class="wrdu-floating wrdu-dark header">
          <h2 class="text">Inner</h2>
          <h2 class="text">Outer</h2>
        </div>
        <div class="wrdu-floating parts">
          <div class="part" v-for="(layer, index) in layers">
            <label :for="'inner' + index" class="layer wrdu-element">
              <input
                :name="'inner' + index"
                :id="'inner' + index"
                type="checkbox"
                v-model="layer.inner"
              />
              <div class="text">{{ layer.name }}</div>
            </label>
            <label :for="'outer' + index" class="layer wrdu-element">
              <input
                :name="'outer' + index"
                :id="'outer' + index"
                type="checkbox"
                v-model="layer.outer"
              />
              <div class="text">{{ layer.name }}</div>
            </label>
          </div>
        </div>
      </div>
      <Tabs
        v-model="model.value"
        :options="[
          { value: 'default', text: 'Default' },
          { value: 'slim', text: 'Slim' },
          { value: 'auto-detect', text: 'Auto' },
        ]"
      />
      <Tabs
        v-model="animation.value"
        :options="[
          { value: 'none', text: 'None' },
          { value: 'idle', text: 'Idle' },
          { value: 'walk', text: 'Walk' },
        ]"
      />
      <div class="hover-area"></div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import Viewer from "~/components/Viewer.vue";

const max = ref(false);

const layers = ref([
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
]);

const model = ref({
  value: "auto-detect",
  text: "Auto",
});

const animation = ref({
  value: "none",
  text: "None",
});

let fileUpdateInterval: ReturnType<typeof setInterval> | null = null;

const skin = ref("/skin.png");

const viewerRef = ref<InstanceType<typeof Viewer> | null>(null);

function screenShot() {
  if (viewerRef.value && viewerRef.value.screenShot) {
    viewerRef.value.screenShot();
  }
}

function reset() {
  if (fileUpdateInterval !== null) {
    clearInterval(fileUpdateInterval);
    fileUpdateInterval = null;
  }

  max.value = false;

  layers.value = [
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
  ];

  model.value = {
    value: "auto-detect",
    text: "Auto",
  };

  animation.value = {
    value: "none",
    text: "None",
  };

  skin.value = "/skin.png";

  if (viewerRef.value && viewerRef.value.reset) {
    viewerRef.value.reset();
  }
}

onMounted(() => {
  const upload = document.getElementById("upload");
  if (upload) {
    upload.addEventListener("click", async () => {
      const fileHandle = await getFileHandle();
      if (fileHandle) {
        await monitorFile(fileHandle);
      }
    });
  }
});

async function getFileHandle() {
  const [fileHandle] = await window.showOpenFilePicker({
    types: [
      {
        description: "Image Files",
        accept: {
          "image/png": [".png"],
        },
      },
    ],
    excludeAcceptAllOption: true,
    multiple: false,
  });
  return fileHandle;
}

async function readFile(fileHandle: FileSystemFileHandle) {
  const file = await fileHandle.getFile();
  const url = URL.createObjectURL(file);
  return url;
}

async function monitorFile(fileHandle: FileSystemFileHandle) {
  let lastUrl = await readFile(fileHandle);

  skin.value = lastUrl;

  if (fileUpdateInterval !== null) {
    clearInterval(fileUpdateInterval);
  }

  fileUpdateInterval = setInterval(async () => {
    const currentUrl = await readFile(fileHandle);
    if (currentUrl !== lastUrl) {
      URL.revokeObjectURL(lastUrl);
      skin.value = currentUrl;
      lastUrl = currentUrl;
    }
  }, 2000);
}
</script>

<style lang="scss">
@import url("/assets/styles/global.scss");

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

  .context {
    position: absolute;
    top: 10px;
    left: 10px;
    max-width: 400px;
    z-index: 1000;
    color: #ffffff;
    opacity: 0.5;
    user-select: none;
    pointer-events: none;
  }

  .overlay {
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    z-index: 100;
    pointer-events: none;
    display: flex;
    justify-content: space-between;
    color: white;
    font-family: Inter;

    .left,
    .right,
    .center {
      display: flex;
      flex-direction: column;
      gap: 20px;
      justify-content: center;
      min-width: 320px;
      width: 320px;
      transition: translate 0.2s ease-in-out;
      position: relative;
      padding: 20px;

      .hover-area {
        display: none;
      }

      &.hidden {
        &.left {
          translate: -330px 0;

          .hover-area {
            right: -60px;
            width: 60px;
            height: 100%;
          }
        }

        &.center {
          height: max-content;
          translate: 0 130px;
          align-self: end;

          .hover-area {
            top: -60px;
            width: 100%;
            height: 60px;
          }
        }

        &.right {
          translate: 330px 0;

          .hover-area {
            left: -60px;
            width: 60px;
            height: 100%;
          }
        }

        &.left,
        &.right,
        &.center {
          .hover-area {
            display: block;
            position: absolute;
          }

          &:has(.hover-area:hover),
          &:hover {
            translate: 0 0;
            pointer-events: all;
          }
        }
      }

      * {
        pointer-events: auto;
      }
    }

    .center {
      justify-content: end;
      align-items: center;

      .actions {
        padding: 5px;
        display: flex;
        gap: 5px;

        .action {
          transition: 0.1s ease-in-out;
          padding: 10px;
          width: 58px;
          height: 58px;
          display: flex;
          align-items: center;
          justify-content: center;
          border-radius: 19px;
          cursor: pointer;
          position: relative;
          overflow: visible;

          .hover {
            overflow: hidden;
            position: absolute;
            width: 200px;
            padding: 10px;
            border-radius: 12px;
            background: rgba(0, 0, 0, 0.6);
            backdrop-filter: blur(20px);
            bottom: 130%;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 3px;
            text-align: center;
            pointer-events: none;
            z-index: 100;
            scale: 0;
            transition: 0.2s ease-in-out;
            transform-origin: center bottom;

            h3 {
              font-size: 14px;
              pointer-events: none;
            }

            p {
              font-size: 12px;
              pointer-events: none;
            }
          }

          &:hover {
            .hover {
              scale: 1;
            }
          }
        }
      }
    }

    .header {
      display: flex;
      justify-content: space-around;
      color: #ffffffcc;
    }

    .parts {
      display: flex;
      flex-direction: column;
      gap: 15px;

      .part {
        display: flex;
        gap: 15px;

        .layer {
          width: 100%;
          display: flex;
          padding: 10px;
          justify-content: center;

          input {
            display: none;
          }
        }
      }
    }

    .skin {
      display: flex;
      flex-direction: column;
      gap: 15px;

      .image {
        padding: 20px;
        border-radius: 9px;

        img {
          image-rendering: pixelated;
          width: 100%;
          height: 100%;
        }
      }

      .upload {
        display: flex;
        gap: 15px;
        align-items: center;
        overflow: hidden;

        button {
          padding: 10px;
          padding-inline: 24px;
          outline: none;
          border: none;
          cursor: pointer;
          color: white;
          background: rgba(250, 250, 250, 0.2);
          transition: 0.1s ease-in-out;
          border-radius: 11px;
          width: 100%;
          font-size: 16px;

          &:hover {
            background: rgba(250, 250, 250, 0.4);
          }
        }
      }
    }
  }
}
</style>
