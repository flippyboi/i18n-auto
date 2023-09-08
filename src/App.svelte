<script lang="ts">
  import axios from "axios";
  import { flatten, unflatten } from "flat";

  enum Lang {
    RU = "ru",
    EN = "en",
    KR = "ky",
    AR = "ar",
    TR = "tr",
    UA = "uk",
    UZ = "uz",
  }

  let text: string;
  let output: string;
  let obj = {};

  const translate = async (language: Lang) => {
    eval("obj=" + text);
    const flat: {} = flatten(obj);
    const flatArr = Object.values(flat);
    const textToTranslate = flatArr.join(" | ");

    const encodedParams = new URLSearchParams();
    encodedParams.set("source_language", "auto");
    encodedParams.set("target_language", language);
    encodedParams.set("text", textToTranslate);

    const options = {
      method: "POST",
      url: "https://text-translator2.p.rapidapi.com/translate",
      headers: {
        "content-type": "application/x-www-form-urlencoded",
        "X-RapidAPI-Key": import.meta.env.VITE_TRANSLATOR_KEY,
        "X-RapidAPI-Host": "text-translator2.p.rapidapi.com",
      },
      data: encodedParams,
    };

    await axios
      .request(options)
      .then(({ data }) => {
        const res = data.data.translatedText.split(" | ");
        let i = 0;
        for (let key in flat) {
          flat[key] = res[i];
          i++;
        }
        output = JSON.stringify(unflatten(flat), null, "\t").replace(
          /"([^"]+)":/g,
          "$1:"
        );
      })
      .catch((e) => console.log(e));
  };
</script>

<main>
  <h1>i18n translator</h1>
  <div class="form">
    <textarea
      class="input"
      placeholder="js object to translate"
      bind:value={text}
    />
    <div class="variants">
      {#each Object.values(Lang) as lng}
        <button on:click={() => translate(lng)}>{lng}</button>
      {/each}
    </div>
  </div>
  <textarea class="output" bind:value={output} placeholder="result" />
</main>

<style>
  .input,
  .output {
    width: 400px;
    height: 300px;
  }

  .variants {
    display: flex;
    gap: 5px;
    flex-basis: 100%;
  }

  .form {
    display: flex;
    flex-direction: column;
    gap: 10px;
    margin-bottom: 10px;
  }
</style>
