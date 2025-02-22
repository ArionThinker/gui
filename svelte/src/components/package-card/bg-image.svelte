<script lang="ts">
  import type { GUIPackage } from "$libs/types";
  import { packagesStore } from "$libs/stores";

  let clazz = "";
  export { clazz as class };

  export let layout: "bottom" | "right" | "left" | "none" = "bottom";

  export let pkg: GUIPackage;
  export let plainImg = false;

  const defaultImgUrl = "/images/default-thumb.jpg";
  $: loadedImg = "";
  let loaded = false;
  let lastProcessedPkg: GUIPackage | null = null;

  const loadImage = async (url: string, force = false): Promise<string> => {
    if (url.includes("cached_images") || force) {
      loadedImg = url;
      loaded = true;
      return url;
    }

    const image = new Image();
    image.src = url;
    return new Promise((resolve, reject) => {
      image.onload = () => {
        loadedImg = url;
        setTimeout(() => {
          loaded = true;
        }, 100);
        resolve(url);
      };
      image.onerror = () => {
        reject(new Error(`file/url does not exist ${url}`));
        loadedImg = defaultImgUrl;
      };
    });
  };

  const getCache = async () => {
    if (pkg.image_512_url) {
      const ts = +new Date();
      loadImage(pkg.image_512_url + `?ts=${ts}`, true).finally(async () => {
        const nextImage = await packagesStore.cachePkgImage(pkg);
        loadImage(nextImage + `?ts=${ts}`, true);
      });
    }
  };

  $: {
    // #706 issue: invalid image is being loaded on pkg list this is just a test fix
    const invalidImage =
      loadedImg && !(loadedImg.includes(pkg.full_name) || loadedImg.includes("localplaceholder"));
    if ((pkg && pkg?.slug !== lastProcessedPkg?.slug) || invalidImage) {
      loaded = false;
      loadedImg = "";
      lastProcessedPkg = pkg;
      getCache();
    }
  }
</script>

{#if plainImg}
  <img alt={pkg.name} src={loadedImg} />
{:else}
  <section class="overflow-hidden bg-black {clazz} {layout}">
    <i
      class="logo icon-tea-logo-iconasset-1 text-gray text-3xl {layout}"
      class:animate-pulse={!pkg.image_added_at}
    />
    <div
      class="opacity-0 transition-all duration-500 {layout}"
      class:opacity-100={loaded}
      style="background-image: url({loadedImg})"
    >
      <!-- dup image: save processing power instead of computing the blur across all the HTML layers -->
      {#if layout !== "none"}
        <aside
          class="blur-sm {layout} opacity-0 transition-all duration-500"
          class:opacity-100={loaded}
        >
          <figure class={layout} style="background-image: url({loadedImg})" />
        </aside>
      {/if}
    </div>
  </section>
{/if}

<style>
  section {
    width: 100%;
    height: 100%;
  }

  .logo {
    position: absolute;
    width: 30px;
    height: 30px;
    margin-left: -15px;
  }
  .logo.none {
    left: 50%;
    top: 50%;
  }
  .logo.bottom {
    left: 50%;
    top: 30%;
  }
  .logo.right {
    left: 22%;
    top: 50%;
    margin-top: -15px;
  }
  .logo.left {
    left: 70%;
    top: 50%;
    margin-top: -15px;
  }

  div {
    position: absolute;
    left: 0px;
    bottom: 0px;
    width: 100%;
    height: 100%;
    background-size: cover;
    box-sizing: border-box;
    background-repeat: no-repeat;
    background-position: 50% 50%;
  }

  div.left {
    background-position: 200px 50%;
  }
  div.right {
    background-repeat: repeat;
    background-position: -250px 50%;
  }
  div.bottom {
    background-repeat: repeat;
    background-position: 50% -70px;
  }
  aside {
    position: absolute;
    bottom: 0px;
    width: 100%;
    overflow: hidden;
  }
  aside.bottom {
    left: 0px;
    height: 50%;
  }

  aside.left {
    left: 0px;
    height: 100%;
    width: 60%;
  }

  aside.right {
    height: 100%;
    right: 0px;
    width: 60%;
  }

  figure {
    position: absolute;
    bottom: 0px;
    width: 100%;
    height: 338px;
    background-size: cover;
    background-repeat: no-repeat;
    background-position: 50% 50%;
  }

  figure.left {
    background-repeat: repeat;
    background-position: 200px 50%;
  }

  figure.right {
    background-repeat: repeat;
    background-position: -250px 50%;
  }
  figure.bottom {
    background-repeat: repeat;
    background-position: 50% -70px;
  }

  aside.bottom figure {
    left: 0px;
  }

  aside.right figure {
    height: 100%;
    /* the overlay is 60% of the image, so we need to oversize the background image back to 100% */
    width: 166.6666666%;
    right: 0px;
  }

  aside.left figure {
    height: 100%;
    /* the overlay is 60% of the image, so we need to oversize the background image back to 100% */
    width: 166.66666666%;
    left: 0px;
  }
</style>
