<template>
  <div>
    <div
      class="
        min-h-1/2
        bg-gray-50
        flex flex-col
        justify-center
        py-12
        sm:px-6
        lg:px-8
      "
    >
      <div class="sm:mx-auto sm:w-full sm:max-w-md">
        <h2 class="mt-6 text-center text-3xl font-extrabold text-gray-900">
          Video aspect ratio editor
        </h2>
        <p class="mt-2 text-center text-sm text-gray-600">
          Upload your video to have it rendered in multiple aspect ratios
        </p>
      </div>

      <div class="mt-8 sm:mx-auto sm:w-full sm:max-w-md">
        <div class="bg-white py-8 px-4 shadow sm:rounded-lg sm:px-10">
          <form
            class="space-y-6"
            action="#"
            method="POST"
            @submit.prevent="submit"
          >
            <div>
              <label
                for="email"
                class="block text-sm font-medium text-gray-700"
              >
                Video
              </label>
              <div class="mt-1">
                <input
                  id="file"
                  name="file"
                  type="file"
                  required
                  @change="handleFile"
                  class="
                    appearance-none
                    block
                    w-full
                    px-3
                    py-2
                    border border-gray-300
                    rounded-md
                    shadow-sm
                    placeholder-gray-400
                    focus:outline-none
                    focus:ring-indigo-500
                    focus:border-indigo-500
                    sm:text-sm
                  "
                />
              </div>
            </div>

            <div>
              <p
                v-if="uploading"
                class="text-center text-sm font-semibold text-gray-700"
              >
                Uploading...
              </p>
              <button
                v-else
                type="submit"
                class="
                  w-full
                  flex
                  justify-center
                  py-2
                  px-4
                  border border-transparent
                  rounded-md
                  shadow-sm
                  text-sm
                  font-medium
                  text-white
                  bg-indigo-600
                  hover:bg-indigo-700
                  focus:outline-none
                  focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500
                "
              >
                Upload
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>

    <ul
      role="list"
      class="
        space-y-12
        sm:grid sm:grid-cols-2
        sm:gap-x-6 sm:gap-y-12
        sm:space-y-0
        lg:grid-cols-3
        lg:gap-x-8
        m-10
      "
    >
      <li v-for="(video, index) in videos" :key="index">
        <div class="space-y-4">
          <div
            class="aspect-w-3 aspect-h-2 flex justify-center"
            v-html="video.element"
          ></div>

          <div class="space-y-2">
            <div class="text-center text-lg leading-6 font-medium space-y-1">
              <h3 class="text-sm font-medium p-2">{{ video.ratio.label }}</h3>
              <a
                target="_blank"
                :href="video.url"
                class="
                  inline-flex
                  items-center
                  px-2.5
                  py-1.5
                  border border-transparent
                  text-xs
                  font-medium
                  rounded
                  text-indigo-700
                  bg-indigo-100
                  hover:bg-indigo-200
                  focus:outline-none
                  focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500
                "
              >
                Download
              </a>
            </div>
          </div>
        </div>
      </li>
    </ul>
  </div>
</template>


<script>
export default {
  data() {
    return {
      uploading: false,
      uploadVideo: null,
      cloudinaryVideo: null,
      aspectRatios: [
        { value: "7680x4320", label: "7680 x 4320 (8K UHDTV)" },
        {
          value: "5120x2880",
          label: "5120 x 2880 (5K, iMac with retina screen)",
        },
        { value: "3840x2160", label: "3840 × 2160 (4K UHDTV)" },
        { value: "2048x1536", label: "2048 x 1536 (iPad with retina screen)" },
        { value: "1920x1200", label: "1920 x 1200 (WUXGA)" },
        { value: "1920x1080", label: "1920 x 1080 (HD TV, iPhone 6 plus)" },
        { value: "1334x750", label: "1334 x 750 (iPhone 6)" },
        { value: "1200x630", label: "1200 x 630 (Facebook)" },
        { value: "1136x640", label: "1136 x 640 (iPhone 5 screen)" },
        { value: "1024x768", label: "1024 x 768 (iPad)" },
        { value: "1024x512", label: "1024 x 512 (Twitter)" },
        { value: "960x640", label: "960 x 640 (iPhone 4 screen)" },
        { value: "800x600", label: "800 x 600" },
        { value: "728x90", label: "728 x 90 (Common web banner ad size)" },
        { value: "720x576", label: "720 x 576 (PAL)" },
        { value: "640x480", label: "640 x 480 (VGA)" },
        { value: "576x486", label: "576 x 486 (NTSC)" },
        { value: "320x480", label: "320 x 480 (HVGA)" },
      ],
      videos: [],
    };
  },
  methods: {
    async handleFile(e) {
      this.uploadVideo = e.target.files[0];
    },
    async readData(f) {
      return new Promise((resolve) => {
        const reader = new FileReader();
        reader.onloadend = () => resolve(reader.result);
        reader.readAsDataURL(f);
      });
    },
    async submit() {
      this.uploading = true;
      const videoData = await this.readData(this.uploadVideo);
      this.cloudinaryVideo = await this.$cloudinary.upload(videoData, {
        upload_preset: "default-preset",
        folder: "nuxtjs-video-aspect-ratio-editor",
      });
      this.uploading = false;
      this.generateVideos();
    },
    generateVideos() {
      this.videos = this.aspectRatios.map((ratio) => ({
        ratio,
        url: this.$cloudinary.video.url(this.cloudinaryVideo.public_id, {
          width: parseInt(ratio.value.split("x")[0]),
          height: parseInt(ratio.value.split("x")[1]),
          crop: "crop",
        }),
        element: this.$cloudinary.video
          .element(this.cloudinaryVideo.public_id, {
            width: parseInt(ratio.value.split("x")[0]),
            height: parseInt(ratio.value.split("x")[1]),
            crop: "crop",
            controls: true,
          })
          .toHtml(),
      }));
    },
  },
};
</script>