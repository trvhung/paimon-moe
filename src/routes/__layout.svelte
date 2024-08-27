<script>
  import '../app.css';

  import { onMount } from 'svelte';
  import { derived } from 'svelte/store';
  import { fade } from 'svelte/transition';
  import localforage from 'localforage';
  import { t } from 'svelte-i18n';
  import { startClient } from '../i18n.js';
  import { navigating, page } from '$app/stores';

  import Modal from 'svelte-simple-modal';
  import { mdiDiscord, mdiFacebook, mdiGithub, mdiReddit, mdiTwitter } from '@mdi/js';

  import Sidebar from '../components/Sidebar/Sidebar.svelte';
  import Header from '../components/Header.svelte';
  import DataSync from '../components/DataSync.svelte';

  import { showSidebar } from '../stores/sidebar';
  import { checkLocalSave } from '../stores/saveManager';
  import TodoData from '../components/TodoData.svelte';
  import SettingData from '../components/SettingData.svelte';
  import Toast from '../components/Toast.svelte';
  import Icon from '../components/Icon.svelte';
  import ServiceWorker from '../components/ServiceWorker.svelte';

  const delayedPreloading = derived(navigating, (_, set) => {
    set(true);
    setTimeout(() => set(true), 250);
  });

  startClient();

  page.subscribe(() => {
    try {
      window.reloadAdSlots();
    } catch (error) {}
  });

  onMount(async () => {
    console.log('localforage config');
    localforage.config({
      driver: [localforage.INDEXEDDB, localforage.LOCALSTORAGE],
      name: 'paimonmoe',
      version: 1.0,
      description: 'paimonmoe local storage',
    });
    window.localforage = localforage;
    await checkLocalSave();
  });

  $: segment = $page.url.pathname.substring(1).split('/')[0];
</script>

<Header />
<Modal>
  <Sidebar {segment} />
  {#if $showSidebar}
    <Sidebar {segment} mobile />
  {/if}
  <DataSync>
    <TodoData />
    <SettingData />
    <Toast />
    <main style="flex: 1 0 auto;">
      <slot />
    </main>
  </DataSync>
  <ServiceWorker />
</Modal>
{#if $navigating && $delayedPreloading}
  <div transition:fade class="loading-bar" />
{/if}
<div class="lg:ml-64 px-4 md:px-8 py-8 flex flex-col md:pb-24">
  <p class="text-gray-400">
    {$t('footer.affliate')}<br />
    {$t('footer.copyright')}
  </p>
  <div class="flex mt-4 md:items-center flex-col md:flex-row">
    <a class="text-gray-400 hover:text-primary" href="https://discord.com/" rel="nofollow" target="_blank">
      <Icon path={mdiDiscord} size={1.5} />
      {$t('footer.discord')}
    </a>
    <div class="text-gray-400 mt-4 md:mt-0 md:ml-4 flex flex-col md:pl-4 md:border-l border-gray-600">
      <span class="text-gray-500">{$t('footer.community')}</span>
      <div>
        <a
          class="text-gray-400 hover:text-primary whitespace-nowrap"
          href=""
          target="_blank"
        >
          <Icon path={mdiGithub} size={1} />
          {$t('footer.link.github')}
        </a>
        <a
          class="text-gray-400 hover:text-primary whitespace-nowrap"
          href=""
          target="_blank"
        >
          <Icon path={mdiTwitter} size={1} />
          {$t('footer.link.devTwitter')}
        </a>
      </div>
    </div>
    <div class="text-gray-400 mt-4 md:mt-0 md:ml-4 flex flex-col md:pl-4 md:border-l border-gray-600">
      <span class="text-gray-500">{$t('footer.official')}</span>
      <svg
      width="36"
      height="36"
      viewBox="0 0 24 24"
      xmlns="http://www.w3.org/2000/svg"
      fill-rule="evenodd"
      clip-rule="evenodd"
      class="fill-current">
      <path
        d="M22.672 15.226l-2.432.811.841 2.515c.33 1.019-.209 2.127-1.23 2.456-1.15.325-2.148-.321-2.463-1.226l-.84-2.518-5.013 1.677.84 2.517c.391 1.203-.434 2.542-1.831 2.542-.88 0-1.601-.564-1.86-1.314l-.842-2.516-2.431.809c-1.135.328-2.145-.317-2.463-1.229-.329-1.018.211-2.127 1.231-2.456l2.432-.809-1.621-4.823-2.432.808c-1.355.384-2.558-.59-2.558-1.839 0-.817.509-1.582 1.327-1.846l2.433-.809-.842-2.515c-.33-1.02.211-2.129 1.232-2.458 1.02-.329 2.13.209 2.461 1.229l.842 2.515 5.011-1.677-.839-2.517c-.403-1.238.484-2.553 1.843-2.553.819 0 1.585.509 1.85 1.326l.841 2.517 2.431-.81c1.02-.33 2.131.211 2.461 1.229.332 1.018-.21 2.126-1.23 2.456l-2.433.809 1.622 4.823 2.433-.809c1.242-.401 2.557.484 2.557 1.838 0 .819-.51 1.583-1.328 1.847m-8.992-6.428l-5.01 1.675 1.619 4.828 5.011-1.674-1.62-4.829z"></path>
    </svg>
    <p>Special thank for  Paimon.Moe</p>
    </div>
    <div
      class="text-gray-400 mt-4 md:mt-0 md:ml-4 flex flex-col justify-center h-full md:pl-4 md:border-l border-gray-600"
    >
      <a class="text-gray-400 hover:text-primary" href="/privacy-policy">{$t('footer.link.privacyPolicy')}</a>
      <!-- svelte-ignore a11y-invalid-attribute -->
      <a class="text-gray-400 hover:text-primary nn-cmp-show" href="#">{$t('footer.link.cookieSettings')}</a>
    </div>
  </div>
</div>

<style lang="postcss">
  .loading-bar {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 3px;
    background-color: transparent;
    overflow: hidden;
    z-index: 100;
  }

  .loading-bar::after {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    width: 20%;
    height: 3px;
    background-color: #4e7cff;
    box-shadow: 10px 0 20px 20px #4e7cff, -10px 0 20px 20px #4e7cff;
    animation: loading-bar 2s cubic-bezier(0.37, 0, 0.63, 1) infinite;
  }

  @keyframes loading-bar {
    0% {
      left: -25%;
    }
    100% {
      left: 110%;
    }
  }
</style>
