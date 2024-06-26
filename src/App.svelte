<script>
  import CopyBut from "./lib/CopyBut.svelte";
  import RVApp from "./lib/RVApp.svelte";

  function getPkgs(patchesJson) {
    const supportedAppsSet = new Set();
    for (const patch of patchesJson) {
      if (patch["compatiblePackages"] !== null) {
        patch["compatiblePackages"]
          .map((e) => e["name"])
          .forEach((e) => {
            supportedAppsSet.add(e);
          });
      }
    }
    const supportedApps = [...supportedAppsSet];
    supportedApps.sort((a, b) => {
      const isPkgA = PKG_APPNAME[a] !== undefined;
      const isPkgB = PKG_APPNAME[b] !== undefined;

      if (isPkgA && isPkgB) return 0;
      else if (isPkgA) return -1;
      else if (isPkgB) return 1;
    });

    return supportedApps;
  }

  async function getPatches(pkgName, patchesJson) {
    const patches = [];
    const universalPatches = [];
    for (const patch of await patchesJson) {
      for (const pkg of patch["compatiblePackages"] || []) {
        if (pkg["name"] === pkgName) {
          const versionsSet = new Set();
          if (pkg["versions"] !== null) {
            pkg["versions"].forEach((e) => {
              versionsSet.add(e);
            });
          }
          const versions = [...versionsSet];
          patches.push({
            ["name"]: patch["name"],
            ["description"]: patch["description"],
            ["pkg_versions"]: versions,
            ["use"]: patch["use"],
            ["patchOptions"]: patch["options"],
          });
        }
      }
      if (patch["compatiblePackages"] === null) {
        universalPatches.push({
          ["name"]: patch["name"],
          ["description"]: patch["description"],
          ["pkg_versions"]: ["Universal patch"],
          ["use"]: patch["use"],
          ["patchOptions"]: patch["options"],
        });
      }
    }
    return patches.concat(universalPatches);
  }

  const PKG_APPNAME = {
    "com.google.android.youtube": "YouTube",
    "tv.twitch.android.app": "Twitch",
    "com.twitter.android": "Twitter",
    "com.laurencedawson.reddit_sync": "Sync",
    "com.laurencedawson.reddit_sync.pro": "Sync Pro",

    "com.google.android.apps.youtube.music": "Music",
    "com.rubenmayayo.reddit": "Boost",
    "com.zhiliaoapp.musically": "Tiktok",
    "com.reddit.frontpage": "Reddit",
    "com.facebook.orca": "Messenger",
    "com.instagram.android": "Instagram",
  };

  function getReprName(pkgName) {
    if (PKG_APPNAME[pkgName] !== undefined) return PKG_APPNAME[pkgName];
    else return pkgName;
  }

  const PKG_APKMIRROR = {
    "com.google.android.youtube":
      "https://apkmirror.com/apk/google-inc/youtube/",
    "tv.twitch.android.app":
      "https://apkmirror.com/apk/twitch-interactive-inc/twitch/",
    "com.twitter.android": "https://apkmirror.com/apk/twitter-inc/twitter/",
    "com.google.android.apps.youtube.music":
      "https://apkmirror.com/apk/google-inc/youtube-music/",
    "com.zhiliaoapp.musically":
      "https://apkmirror.com/apk/tiktok-pte-ltd/tik-tok-including-musical-ly/",
    "com.reddit.frontpage": "https://apkmirror.com/apk/redditinc/reddit/",
    "com.facebook.orca": "https://apkmirror.com/apk/facebook-2/messenger/",
    "com.instagram.android":
      "https://apkmirror.com/apk/instagram/instagram-instagram/",
  };
  function getApkmirror(pkgName) {
    if (PKG_APKMIRROR[pkgName] !== undefined) return PKG_APKMIRROR[pkgName];
    else return undefined;
  }

  let TOML = [];
  let configTOMLVisible = false;

  let patchesJson = fetch(
    "https://raw.githubusercontent.com/revanced/revanced-patches/main/patches.json"
  ).then((r) => r.json());
</script>

<main>
  <p class="m-4">Revanced Config Generator</p>
  <div class="inline-flex">
    <a href="https://t.me/rvc_magisk" target="_blank" rel="noreferrer noopener">
      <svg class="m-4" width="2rem" height="auto" viewBox="0 0 16 16">
        <path
          d="M16 8A8 8 0 1 1 0 8a8 8 0 0 1 16 0zM8.287 5.906c-.778.324-2.334.994-4.666 2.01-.378.15-.577.298-.595.442-.03.243.275.339.69.47l.175.055c.408.133.958.288 1.243.294.26.006.549-.1.868-.32 2.179-1.471 3.304-2.214 3.374-2.23.05-.012.12-.026.166.016.047.041.042.12.037.141-.03.129-1.227 1.241-1.846 1.817-.193.18-.33.307-.358.336a8.154 8.154 0 0 1-.188.186c-.38.366-.664.64.015 1.088.327.216.589.393.85.571.284.194.568.387.936.629.093.06.183.125.27.187.331.236.63.448.997.414.214-.02.435-.22.547-.82.265-1.417.786-4.486.906-5.751a1.426 1.426 0 0 0-.013-.315.337.337 0 0 0-.114-.217.526.526 0 0 0-.31-.093c-.3.005-.763.166-2.984 1.09z"
        />
      </svg>
    </a>

    <a
      href="https://github.com/reforget-id/revanced-config-gen"
      target="_blank"
      rel="noreferrer noopener"
    >
      <svg class="m-4" width="2rem" height="auto" viewBox="0 0 16 16">
        <path
          d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.012 8.012 0 0 0 16 8c0-4.42-3.58-8-8-8z"
        />
      </svg>
    </a>
  </div>

  <br />
  <button
    on:click={() => {
      configTOMLVisible = !configTOMLVisible;
    }}
    class="m-3 mb-5 justify-center items-center space-x-2 border border-gray-300 rounded-md py-2 px-4 bg-white text-sm leading-5 font-medium text-gray-900 hover:text-gray-600 focus:outline-none focus:border-gray-300 focus:ring-blue active:bg-gray-50 active:text-gray-800"
    >{configTOMLVisible ? "Hide Config" : "Create Config"}</button
  >
  {#if configTOMLVisible}
    <div
      class="bg-gray-300 rounded-lg mx-4 my-0 border border-gray-400 p-4 text-sm"
    >
      <CopyBut id={"config-toml-text"} />

      <div class="text-gray-600"># config.toml</div>
      <div id="config-toml-text">
        {#each TOML as t}
          {#if !t.deleted && t.apkmirror_dlurl !== "https://apkmirror.com/apk/" && t.apkmirror_dlurl.length !== 0}
            [{t.app_name}]<br />
            apkmirror-dlurl = "{t.apkmirror_dlurl}"<br />
            {#if t.patches_source !== "revanced/revanced-patches"}
              patches-source = "{t.patches_source}"<br />
              integrations-source = "{t.patches_source.split(
                "/"
              )[0]}/revanced-integrations"<br />
              rv-brand = "{t.patches_source.split("/")[0]} ReVanced"<br />
            {/if}
            {#if t.arch !== "universal"}
              arch = "{t.arch}"<br />
            {/if}
            {#if t.build_mode !== "apk"}
              build-mode = "{t.build_mode}"<br />
            {/if}
            {#if t.version !== "auto"}
              version = "{t.version}"<br />
            {/if}
            {#if t.included_patches !== ""}
              included-patches = "{t.included_patches}"<br />
            {/if}
            {#if t.excluded_patches !== ""}
              excluded-patches = "{t.excluded_patches}"<br />
            {/if}
            {#if t.exclusive_patches}
              exclusive-patches = {t.exclusive_patches}<br />
            {/if}
            <br />
          {/if}
        {/each}
      </div>
    </div>

    <div
      class="mt-3 bg-gray-300 rounded-lg mx-4 my-0 border border-gray-400 p-4 text-sm"
    >
      <CopyBut id={"options-json-text"} />

      <div class="text-gray-600"># options.json</div>
      <div id="options-json-text">
        <pre>
        {(() => {
            let oo = [];
            TOML.forEach((t) => {
              oo = [...oo, ...t.opts];
            });
            return "\n" + JSON.stringify(oo, undefined, 2);
          })()}
        </pre>
      </div>
    </div>
  {/if}

  {#await patchesJson.then((j) => getPkgs(j)) then pkgs}
    {#each pkgs as pkgName, i}
      <RVApp
        {pkgName}
        revancedPatches={getPatches(pkgName, patchesJson)}
        reprName={getReprName(pkgName)}
        apkmirror_dlurl={getApkmirror(pkgName)}
        bind:TOML={TOML[i]}
      />
    {/each}
  {/await}
</main>

<style>
</style>
