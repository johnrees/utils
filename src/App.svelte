<script lang="ts">
  import bs58 from "bs58";
  let input = $state("");

  const options = ["base58", "base64", "uint8array", "utf8"] as const;

  let from = $state<(typeof options)[number]>("base64");
  let to = $state<(typeof options)[number]>("base58");

  const output = $derived.by(() => {
    let invalidInput = false;

    const inputBytes = (() => {
      try {
        switch (from) {
          case "base58":
            return bs58.decode(input);
          case "base64":
            return new Uint8Array(
              atob(input)
                .split("")
                .map((char) => char.charCodeAt(0))
            );
          case "utf8":
            return new TextEncoder().encode(input);
          case "uint8array":
            const arr = JSON.parse(input);
            if (Array.isArray(arr) && arr.every((n) => typeof n === "number")) {
              return Uint8Array.from(arr);
            } else {
              throw new Error("Invalid");
            }
        }
      } catch (err) {
        console.error(err);
        invalidInput = true;
      }
    })();

    if (inputBytes) {
      switch (to) {
        case "base58":
          return bs58.encode(inputBytes);
        case "base64":
          return btoa(String.fromCharCode.apply(null, Array.from(inputBytes)));
        case "utf8":
          return new TextDecoder().decode(inputBytes);
        case "uint8array":
          return JSON.stringify(Array.from(inputBytes));
      }
    } else if (invalidInput) {
      return `(Invalid ${from} input)`;
    }
  });

  $effect(() => {
    if (from === to) {
      to = options.find((o) => o !== from) as typeof to;
    }
  });

  const placeholders = {
    base58: "3dh6vV7BgQ",
    base64: "RXhhbXBsZQ==",
    utf8: "Example",
    uint8array: "[69,120,97,109,112,108,101]",
  } satisfies Record<(typeof options)[number], string>;
</script>

<div class="grid">
  <div class="select-wrapper">
    <label for="from-select">From:</label>
    <select id="from-select" bind:value={from}>
      {#each options as option}
        <option value={option}>{option}</option>
      {/each}
    </select>
  </div>

  <button
    onclick={() => {
      [from, to] = [to, from];
    }}
  >
    Flip
  </button>

  <div class="select-wrapper">
    <label for="to-select">To:</label>
    <select id="to-select" bind:value={to}>
      {#each options as option}
        <option value={option} disabled={option === from}>
          {option}
        </option>
      {/each}
    </select>
  </div>

  <textarea bind:value={input} placeholder={placeholders[from]}></textarea>
  <div></div>
  <textarea value={output} readonly></textarea>
</div>

<style>
  .grid {
    display: grid;
    grid-template-columns: 1fr auto 1fr;
    grid-template-rows: auto 1fr;
    gap: 1rem;
    padding: 1rem;
    flex: 1;
  }

  textarea {
    box-sizing: border-box;
    width: 100%;
    resize: none;
    padding: 0.5rem;
    font-family: var(
      --fontStack-monospace,
      ui-monospace,
      SFMono-Regular,
      SF Mono,
      Menlo,
      Consolas,
      Liberation Mono,
      monospace
    );
  }

  .select-wrapper {
    display: flex;
    flex-direction: row;
    align-items: center;
    label {
      margin-right: 0.5rem;
    }
    select {
      width: 100%;
      padding: 0.2rem;
    }
  }
</style>
