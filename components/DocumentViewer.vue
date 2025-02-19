<template>
  <div
    id="content-container"
    ref="ducumentContainerRef"
    class="flex flex-col justify-start items-start px-4 lg:pr-64 w-full h-auto relative overflow-x-hidden overflow-y-auto"
  >
    <div
      id="content"
      class="flex flex-col justify-start items-center w-full mx-auto lg:max-w-3xl 2xl:max-w-4xl"
    >
      <div class="w-full markdown-body nuxt-content pt-6">
        <h1>{{ document.title }}</h1>
      </div>
      <nuxt-content
        class="w-full pb-6 pt-4 markdown-body"
        :document="document"
      />
      <div
        class="w-full flex flex-col sm:flex-row sm:justify-start sm:items-center text-base"
      >
        <a
          class="py-1 flex flex-row justify-start items-center text-gray-600 hover:text-accent"
          :href="`https://github.com/bytebase/bytebase.com/blob/main${githubFilePath}`"
          target="_blank"
        >
          Edit this page on GitHub
          <img
            class="h-4 ml-2"
            src="~/assets/svg/external-link.svg"
            alt="prev"
          />
        </a>
      </div>
      <div
        class="w-full mt-4 pb-12 pt-4 flex flex-row justify-between border-t border-gray-200"
      >
        <nuxt-link
          v-if="prev"
          :to="localePath(`/docs${prev.path}`)"
          class="py-2 flex flex-row justify-start items-center text-base text-gray-600 hover:text-accent"
        >
          <span>{{ "← " + prev.title }}</span>
        </nuxt-link>
        <span v-else></span>
        <nuxt-link
          v-if="next"
          :to="localePath(`/docs${next.path}`)"
          class="py-2 flex flex-row justify-end items-center text-base text-gray-600 hover:text-accent"
        >
          <span>{{ next.title + " →" }}</span>
        </nuxt-link>
        <span v-else></span>
      </div>
    </div>

    <!-- TOC -->
    <div
      class="hidden fixed right-0 top-32 pt-12 w-60 py-2 h-auto max-h-screen flex-shrink-0 lg:flex flex-col justify-start items-start overflow-y-auto text-sm"
    >
      <Toc :content="document" :scroll-offset="20" class="md:flex" />
    </div>

    <!-- Subscribtion popup -->
    <div
      v-if="false"
      class="transition-all fixed bottom-0 left-0 w-full bg-white z-10 lg:px-36 border-t shadow"
      :style="{ 'margin-bottom': state.showSubscribtionPopup ? '0' : '-100%' }"
    >
      <span
        class="absolute top-2 right-2 lg:right-44 mr-2 cursor-pointer text-lg opacity-80 hover:opacity-60"
        @click="onCloseSubscribtionPopUp"
        ><img src="~/assets/svg/x.svg" class="w-6 h-auto" alt="" />
      </span>
      <SubscribeSection
        class="border-none"
        :module-name="'subscribe.docs'"
        @subscribed="onSubscribed"
      />
    </div>
  </div>
</template>

<script lang="ts">
import {
  defineComponent,
  onMounted,
  reactive,
  ref,
  useFetch,
  useContext,
  useMeta,
  watch,
} from "@nuxtjs/composition-api";
import dayjs from "dayjs";
import relativeTime from "dayjs/plugin/relativeTime";
import { ContentDocument } from "~/types/docs";
import storage from "~/common/storage";
import { validDocsCategoryList } from "~/common/const";
import SubscribeSection from "./SubscribeSection.vue";
import Toc from "./Toc.vue";

dayjs.extend(relativeTime);

export default defineComponent({
  components: { SubscribeSection, Toc },
  props: {
    document: Object,
  },
  setup(props) {
    const { $content, route } = useContext();
    const meta = useMeta({});
    const state = reactive({
      showSubscribtionPopup: false,
    });
    const ducumentContainerRef = ref<HTMLDivElement>();
    const prev = ref<any>(undefined);
    const next = ref<any>(undefined);
    const githubFilePath = `/content${props.document?.path}${props.document?.extension}`;

    useFetch(async () => {
      const locale = "en";
      const category = route.value.params.category;
      const layout = (await $content(
        "docs",
        locale,
        validDocsCategoryList.includes(category) ? category : "",
        "_layout"
      ).fetch()) as any as ContentDocument;
      const nodes = layout.body.children
        .filter((n) => n.tag === "h2" || n.tag === "h3" || n.tag === "h4")
        .map((n) => {
          let level = 1;
          if (n.tag === "h3") {
            level = 2;
          } else if (n.tag === "h4") {
            level = 3;
          }

          const child = n.children[1];
          let path = undefined;
          let title = "";
          if (child.type !== "text") {
            path = child.props.to;
            title = child.children[0].value;
          } else {
            title = child.value;
          }

          return {
            level,
            title,
            path,
          };
        })
        .filter((n) => n.path !== undefined);

      const index = nodes.findIndex((n) =>
        props?.document?.path.endsWith(n.path)
      );
      prev.value = nodes[index - 1];
      next.value = nodes[index + 1];
    });

    onMounted(() => {
      // Dynamicly set metadata.
      meta.title.value = props.document?.title || "Bytebase Document";
      meta.meta.value = [
        {
          hid: "description",
          name: "description",
          content:
            props.document?.description ||
            (ducumentContainerRef.value?.textContent || "")
              ?.replaceAll(/\r?\n/g, " ")
              .replaceAll(/\s+/g, " ")
              .slice(meta.title.value?.length, 128)
              .trim() + "...",
        },
      ];

      // Add the `Copy` button for pre elements without plain language.
      const preElementNodeList = ducumentContainerRef.value?.querySelectorAll(
        "pre:not(.language-plain)"
      );
      if (preElementNodeList && preElementNodeList.length > 0) {
        const preElementList = Array.from(preElementNodeList);
        for (const preElement of preElementList) {
          const copyBtn = document.createElement("button");
          copyBtn.innerText = "Copy";
          copyBtn.className = "copy-btn";
          preElement.parentElement?.appendChild(copyBtn);
          copyBtn.addEventListener("click", async () => {
            if (navigator.clipboard) {
              const text = (preElement as HTMLElement).innerText;
              await navigator.clipboard.writeText(text);
            }
            copyBtn.innerText = "Copied";
            setTimeout(() => {
              copyBtn.innerText = "Copy";
            }, 2000);
          });
        }
      }
    });

    watch(route, () => {
      if (ducumentContainerRef.value) {
        const targetEl = ducumentContainerRef.value.querySelector(
          `a[href='${route.value.hash}']`
        ) as HTMLAnchorElement;
        targetEl?.click();
      }
    });

    const onCloseSubscribtionPopUp = () => {
      state.showSubscribtionPopup = false;
      storage.set({
        hasShownSubscribtionPopupInDocs: true,
      });
    };

    const onSubscribed = () => {
      storage.set({
        hasShownSubscribtionPopupInDocs: true,
      });
    };

    return {
      state,
      prev,
      next,
      ducumentContainerRef,
      githubFilePath,
      onCloseSubscribtionPopUp,
      onSubscribed,
    };
  },
  head: {},
});
</script>

<style>
.nuxt-content .copy-btn {
  @apply absolute top-0.5 right-0.5 text-xs px-1 italic bg-gray-200 rounded opacity-60 hover:opacity-100;
}
</style>

<style scoped>
@import "~/assets/css/github-markdown-style.css";

.nuxt-content .nuxt-content-highlight {
  @apply relative;
}
.nuxt-content .nuxt-content-highlight pre.language-bash code::before {
  @apply text-gray-400;
  content: "$ ";
}
.nuxt-content h2 > a:first-child:before,
.nuxt-content h3 > a:first-child:before {
  @apply text-blue-600 text-xl leading-6 font-light;
  content: "#";
  margin-left: -1.25rem;
  padding-right: 0.5rem;
  position: absolute;
  visibility: hidden;
}
.nuxt-content h2 > a:first-child:before {
  @apply leading-8;
}
.nuxt-content h2:hover a:first-child:before,
.nuxt-content h3:hover a:first-child:before {
  visibility: visible;
}
</style>
