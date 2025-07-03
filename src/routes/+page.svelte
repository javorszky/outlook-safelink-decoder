<script>
  let input = $state('');
  let qp = $state([]);
  let scheme = $state('');
  let domain = $state('');
  let path = $state('');
  let fullURL = $state('');
  let done = $state(false);

  class ParsedURL {
    input

    parsedScheme
    parsedDomain
    parsedPath
    parsedQueryParams

    constructor(text) {
      this.input = text;

      let inputUrlObj;
      try {
        inputUrlObj = new URL(this.input);
      } catch {
        throw new Error("Invalid URL format");
      }

      // Second check, if we're not using https, then bail. The safelink URL
      // always needs to be https.
      if (inputUrlObj.protocol !== 'https:') {
        throw new Error("URL must use HTTPS");
      }


      // Third check: if the URL is not an outlook safelink, bail, because we
      // only care about outlook safelinks. 
      if (!inputUrlObj.hostname.endsWith('.safelinks.protection.outlook.com')) {
        throw new Error("URL must be from Outlook's safelinks domain");
      }

      // Gather all of the query parameters, and store them in an object, and
      // separate out the original URL that's in the ?url= query param.
      let originalQueryParams = {};
      let originalURL = "";

      for (const [key, value] of inputUrlObj.searchParams.entries()) {
        if (key !== 'url') {
          originalQueryParams[key] = value;
        } else {
          originalURL = value;
        }
      }

      let url;
      try {
        url = new URL(originalURL);
      } catch (error) {
        throw new Error("Invalid URL format for the original URL")
      }

      this.parsedScheme = url.protocol;
      scheme = url.protocol;

      const collectedParams = new URLSearchParams(originalQueryParams);

      this.parsedDomain = url.hostname;
      domain = url.hostname;

      this.parsedPath = url.pathname;
      path = url.pathname;

      let href = scheme + '//' + this.parsedDomain + this.parsedPath + '?' + collectedParams.toString();
      fullURL = href;

      

      this.parsedQueryParams = originalQueryParams;
      qp = originalQueryParams;

      done = true;
    }

    reset = function() {
      this.input = '';
      input = '';

      this.parsedScheme = '';
      scheme = '';

      this.parsedDomain = '';
      domain = '';

      this.parsedPath = '';
      path = '';

      this.parsedQueryParams = '';
      qp = [];

      fullUrl = '';

      done = false;
    }
  }
</script>

<div>
  <label for="url" class="block text-sm/6 font-medium text-gray-900">Safelink URL</label>
  <div class="mt-2">
    <input bind:value={input} type="url" name="url" id="url" class="block w-full rounded-md bg-white px-3 py-1.5 text-base text-gray-900 outline-1 -outline-offset-1 outline-gray-300 placeholder:text-gray-400 focus:outline-2 focus:-outline-offset-2 focus:outline-indigo-600 sm:text-sm/6" placeholder="https://*.safelink.outlook.com/..." />

    <button type="button" class="mt-10 inline-flex items-center gap-x-2 rounded-md bg-indigo-600 px-3.5 py-2.5 text-sm font-semibold text-white shadow-xs hover:bg-indigo-500 focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600 cursor-pointer" onclick={() => new ParsedURL(input)}>
      Parse
      <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
          <path stroke-linecap="round" stroke-linejoin="round" d="M15.75 9V5.25A2.25 2.25 0 0 0 13.5 3h-6a2.25 2.25 0 0 0-2.25 2.25v13.5A2.25 2.25 0 0 0 7.5 21h6a2.25 2.25 0 0 0 2.25-2.25V15m3 0 3-3m0 0-3-3m3 3H9" />
      </svg>
    </button>

    {#if done}
      <div class="mt-6 border-t border-gray-100">
        <dl class="divide-y divide-gray-100">
          <div class="px-4 py-6 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-0">
            <dt class="text-sm/6 font-medium text-gray-900">Scheme</dt>
            <dd class="mt-1 text-sm/6 text-gray-700 sm:col-span-2 sm:mt-0">{scheme}</dd>
          </div>
          <div class="px-4 py-6 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-0">
            <dt class="text-sm/6 font-medium text-gray-900">Domain</dt>
            <dd class="mt-1 text-sm/6 text-gray-700 sm:col-span-2 sm:mt-0">{domain}</dd>
          </div>
          <div class="px-4 py-6 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-0">
            <dt class="text-sm/6 font-medium text-gray-900">Path</dt>
            <dd class="mt-1 text-sm/6 text-gray-700 sm:col-span-2 sm:mt-0">{path}</dd>
          </div>
        </dl>
      </div>
    {/if}

    {#if Object.keys(qp).length}

      <div class="inline-block min-w-full py-2 align-middle">
        <div class="overflow-hidden shadow-sm ring-1 ring-black/5 sm:rounded-lg">
          <table class="table-auto min-w-full divide-y divide-gray-300 my-0">
            <thead class="bg-gray-50">
              <tr>
                <th scope="col" class="py-3.5 pr-3 pl-4 text-left text-sm font-semibold text-gray-900 sm:pl-6">Query param name</th>
                <th scope="col" class="px-3 py-3.5 text-left text-sm font-semibold text-gray-900">Value</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-gray-200 bg-white">

              {#each Object.entries(qp) as [key, value]}

                <tr>
                  <td class="py-4 pr-3 pl-4 text-sm font-medium whitespace-nowrap text-gray-900 sm:pl-6">{key}</td>
                  <td class="px-3 py-4 text-sm wrap-anywhere text-gray-500">{value}</td>
                </tr>

              {/each}

            </tbody>
          </table>
        </div>
      </div>

    {/if}

    {#if done}
      <div class="mx-auto max-w-max">
        <!--
        I will return to this because ugh
        
        <p class="my-2 ">Full link:</p>
        <pre><code class="wrap-anywhere overflow-auto">{fullURL}</code></pre> -->
        
        <a href="{fullURL}" target="_blank" class="inline-flex items-center gap-x-2 rounded-md bg-indigo-600 px-3.5 py-2.5 text-sm font-semibold text-white shadow-xs hover:bg-indigo-500 focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600">Open link in a new tab</a>
      </div>
    {/if}

    <h2>What is this?</h2>
    <p>A strictly client side parser that takes a long safe link that outlook in enterprise environments replaces original links with so it can do its own telemetry and checking for malicious sites in the background.</p>
    <p>This tool extracts the original URL so you know when you're going to click the safe link where you're going to land.</p>
    <h2>Who made it?</h2>
    <p><a href="https://bsky.app/profile/javorszky.rocks">Gabor Javorszky</a></p>
    <h2>Can I see the source code?</h2>
    <p>Yep, <a href="https://github.com/javorszky/outlook-safelink-decoder">it's on GitHub.</a></p>
    <h2>Any technical details?</h2>
    <p>It's made with SvelteKit 5, deployed to a Cloudflare worker.</p>
    <p><i>Copyright © Gabor Javorszky {new Date().getFullYear()}</i></p>
  </div>
</div>
