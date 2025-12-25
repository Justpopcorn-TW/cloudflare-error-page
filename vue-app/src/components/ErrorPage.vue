<template>
  <head>
    <meta property="og:title" content= {{ htmlTitle }} />
    <meta property="og:type" content="website" />
    <meta property="og:url" content={{ customDomain }} />

  </head>
  <div id="cf-wrapper">
    <div id="cf-error-details" class="p-0">
      <header class="mx-auto pt-10 lg:pt-6 lg:px-8 w-240 lg:w-full mb-8">
        <h1 class="inline-block sm:block sm:mb-2 font-light text-60 lg:text-4xl text-black-dark leading-tight mr-2">
          <span class="inline-block" style="margin-right: 10px">{{ title }}</span>
          <span class="code-label">Error code {{ errorCode }}</span>
        </h1>
        <div v-if="!moreInfo.hidden">
          Visit <a :href="moreInfo.link || 'https://www.cloudflare.com/'" target="_blank" rel="noopener noreferrer">{{ moreInfo.text || 'cloudflare.com' }}</a> for more information.
        </div>
        <div class="mt-3">{{ time }}</div>
      </header>

      <div class="my-8 bg-gradient-gray">
        <div class="w-240 lg:w-full mx-auto">
          <div class="clearfix md:px-8">
            <div 
              v-for="itemId in items" 
              :key="itemId" 
              :id="`cf-${itemId}-status`" 
              :class="[
                params.error_source === itemId ? 'cf-error-source' : '', 
                'relative w-1/3 md:w-full py-15 md:p-0 md:py-8 md:text-left md:border-solid md:border-0 md:border-b md:border-gray-400 overflow-hidden float-left md:float-none text-center'
              ]"
            >
              <div class="relative mb-10 md:m-0">
                <span :class="`cf-icon-${getIcon(itemId)} block md:hidden h-20 bg-center bg-no-repeat`"></span>
                <span :class="`cf-icon-${getStatus(itemId)} w-12 h-12 absolute left-1/2 md:left-auto md:right-0 md:top-0 -ml-6 -bottom-4`"></span>
              </div>
              <span class="md:block w-full truncate">{{ getLocation(itemId) }}</span>
              <h3 class="md:inline-block mt-3 md:mt-0 text-2xl text-gray-600 font-light leading-1.3">{{ getName(itemId) }}</h3>
              {{ ' ' }}
              <span class="leading-1.3 text-2xl" :style="{ color: getTextColor(itemId) }">{{ getStatusText(itemId) }}</span>
            </div>
          </div>
        </div>
      </div>

      <div class="w-240 lg:w-full mx-auto mb-8 lg:px-8">
        <div class="clearfix">
          <div class="w-1/2 md:w-full float-left pr-6 md:pb-10 md:pr-0 leading-relaxed">
            <h2 class="text-3xl font-normal leading-1.3 mb-4">What happened?</h2>
            <div v-html="params.what_happened || '<p>There is an internal server error on Cloudflare\'s network.</p>'" />
          </div>
          <div class="w-1/2 md:w-full float-left leading-relaxed">
            <h2 class="text-3xl font-normal leading-1.3 mb-4">What can I do?</h2>
            <div v-html="params.what_can_i_do || '<p>Please try again in a few minutes.</p>'" />
          </div>
        </div>
      </div>

      <div class="cf-error-footer cf-wrapper w-240 lg:w-full py-10 sm:py-4 sm:px-8 mx-auto text-center sm:text-left border-solid border-0 border-t border-gray-300">
        <p class="text-13">
          <span class="cf-footer-item sm:block sm:mb-1">Cloudflare Ray ID: <strong class="font-semibold">{{ rayId }}</strong></span>
          <span class="cf-footer-separator sm:hidden"> &bull; </span>
          <span id="cf-footer-item-ip" class="cf-footer-item sm:block sm:mb-1">
            Your IP:
            {{ ' ' }}
            <button v-if="!showIp" type="button" id="cf-footer-ip-reveal" class="cf-footer-ip-reveal-btn" @click="showIp = true">Click to reveal</button>
            <span v-else id="cf-footer-ip">{{ clientIp }}</span>
            <span class="cf-footer-separator sm:hidden"> &bull; </span>
          </span>
          <span class="cf-footer-item sm:block sm:mb-1"><span>Performance &amp; security by</span> <a rel="noopener noreferrer" :href="perfSecBy.link || 'https://www.cloudflare.com/'" id="brand_link" target="_blank">{{ perfSecBy.text || 'Cloudflare' }}</a></span>
        </p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import '../styles/main.css';

const props = defineProps({
  params: {
    type: Object,
    default: () => ({})
  }
});

const showIp = ref(false);

const errorCode = computed(() => props.params.error_code || 500);
const title = computed(() => props.params.title || 'Internal server error');
const time = computed(() => props.params.time || new Date().toISOString().replace('T', ' ').slice(0, 19) + ' UTC');
const rayId = computed(() => props.params.ray_id || crypto.randomUUID().replace(/-/g, '').slice(0, 16));
const clientIp = computed(() => props.params.client_ip || '1.1.1.1');
const customDomain = computed(() => props.params.domain || (typeof window !== 'undefined' ? window.location.hostname : 'example.com'));
const htmlTitle = computed(() => props.params.html_title || `${customDomain.value} | ${errorCode.value}: ${title.value}`);
const moreInfo = computed(() => props.params.more_information || {});
const perfSecBy = computed(() => props.params.perf_sec_by || {});

const items = ['browser', 'cloudflare', 'host'];

function getIcon(itemId) {
  if (itemId === 'browser') return 'browser';
  if (itemId === 'cloudflare') return 'cloud';
  return 'server';
}

function getDefaultLocation(itemId) {
  if (itemId === 'browser') return 'You';
  if (itemId === 'cloudflare') return 'San Francisco';
  return customDomain.value;
}

function getDefaultName(itemId) {
  if (itemId === 'browser') return 'Browser';
  if (itemId === 'cloudflare') return 'Cloudflare';
  return 'Host';
}

function getStatus(itemId) {
  const item = props.params[`${itemId}_status`] || {};
  return item.status || 'ok';
}

function getTextColor(itemId) {
  const item = props.params[`${itemId}_status`] || {};
  if (item.status_text_color) return item.status_text_color;
  
  const status = item.status || 'ok';
  if (status === 'ok') return '#9bca3e';
  if (status === 'error') return '#bd2426';
  return '#9bca3e';
}

function getStatusText(itemId) {
  const item = props.params[`${itemId}_status`] || {};
  const status = item.status || 'ok';
  return item.status_text || (status === 'ok' ? 'Working' : 'Error');
}

function getLocation(itemId) {
  const item = props.params[`${itemId}_status`] || {};
  return item.location || getDefaultLocation(itemId);
}

function getName(itemId) {
  const item = props.params[`${itemId}_status`] || {};
  return item.name || getDefaultName(itemId);
}

// Set document title
onMounted(() => {
  document.title = htmlTitle.value;
});
</script>
