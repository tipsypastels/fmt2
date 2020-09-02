<script lang="ts">
  const PIN = /(.+)\s*\n\s*pinned\s*\n\s*a message\s*\n\s*to this channel.\s*\n\s*See all the pins\.\s*\n/gm;
  const MSG_START = /^\s*(.*)((?:\d{2}\/\d{2}\/\d{4})|(?:(?:Yesterday|Today) at \d{1,2}:\d{2}(?: [AP]M)?))\s*$/gm;

  const BOLD_FORMATS = [
    {
      name: 'BBCode',
      format: (x: string) => `[b]${x}[/b]`,
    },
    {
      name: 'Markdown',
      format: (x: string) => `**${x}**`,
    },
    {
      name: 'HTML',
      format: (x: string) => `<strong>${x}</strong>`,
    },
  ] as const;

  interface Message {
    content: string;
    author: string;
    dateline: string;
    isBold: boolean;
  };

  function val2Messages(value: string): Message[] {
    let raw = value.replace(PIN, '$1 pinned a message to this channel')
                   .split(MSG_START)
                   .slice(1);

    let results: Message[] = [];
    for (let i = 0; i < raw.length; i += 3) {
      const author   = raw[i].trim();
      const dateline = raw[i + 1].trim();
      const content  = raw[i + 2].trim();

      results.push({ author, dateline, content, isBold: false });
    }

    return results;
  }

  function toggleMessageBold(msgIndex: number) {
    messages = messages.map((msg, i) => {
      if (i === msgIndex) {
        return { ...msg, isBold: !msg.isBold };
      }

      return msg;
    });
  }

  function msg2Str(msg: Message): string {
    if (msg.isBold) {
      return bold(`${msg.author} ${msg.dateline}\n${msg.content}`);
    }

    return `${bold(msg.author)} ${msg.dateline}\n${msg.content}`;
  }

  function copy() {
    const copyTarget = document.createElement('textarea');
    
    copyTarget.classList.add('copy-target');
    copyTarget.value = messages.map(msg2Str).join('\n\n');

    document.body.appendChild(copyTarget);

    copyTarget.focus();
    copyTarget.select();
    copyTarget.setSelectionRange(0, 9999999);

    document.execCommand('copy');

    copyTarget.remove();
    didJustCopy = true;
  }

  function bold(text: string): string {
    return BOLD_FORMATS[boldFormat].format(text);
  }

  let messages: Message[] = [];
  let boldFormat = 0;
  let didJustCopy = false;

  $: mode = messages.length === 0 ? 'input' : 'output';
  
  function setMessagesFromE(e) { 
    messages = val2Messages(e.target.value);
    didJustCopy = false;
  }

  function reset() {
    messages = [];
    didJustCopy = false;
  }

  function advanceBoldFormat() {
    boldFormat = (boldFormat + 1) % BOLD_FORMATS.length;
    didJustCopy = false;
  }
</script>

<h1>
  fmt (v2)

  <span class={`mode mode-${mode}`}>
    {mode} Mode
  </span>
</h1>

<main>
  {#if messages.length === 0}
    <textarea 
      autofocus
      class="input" 
      on:keyup={setMessagesFromE} 
    />
  {:else}
    {#each messages as message, i}
      <article 
        class="message" 
        class:bold={message.isBold}
        on:click={() => toggleMessageBold(i)}
      >
        <strong>{message.author}</strong> {message.dateline}
        <div class="content">{message.content}</div>
      </article>
    {/each}
  {/if}
</main>

<aside class="tip">
  <strong>Tip:</strong> 
  
  {#if mode === 'input'}
    Paste into the box above!
  {:else}
    Click individual messages to toggle highlighting (<button on:click={advanceBoldFormat}>{BOLD_FORMATS[boldFormat].name}</button>). Once you're ready, <button on:click={copy}>click here to copy</button> or <button on:click={reset}>reset fmt</button>.

    {#if didJustCopy}
      <span class="copied">Copied to clipboard</span>
    {/if}
  {/if}
</aside>

<style>
  main {
    border: 2px dashed grey;
    padding: 1rem;
    height: 400px;

    overflow-y: scroll;
    overflow-x: hidden;
    display: flex;
    flex-direction: column;
  }

  @media screen and (min-width: 992px) {
    main {
      height: 600px;
    }
  }

  .tip {
    padding-top: 1rem;
  }

  .input {
    background-color: transparent;
    border: none;
    outline: none;

    flex-grow: 1;
    display: block;
    resize: none;

    padding: 0;
  }

  .input::-webkit-scrollbar {
    display: none;
  }

  :global(.copy-target) {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border: none;
  }

  .message {
    cursor: pointer;
    margin-bottom: 1rem;
    user-select: none;
  }

  .message.bold {
    font-weight: bold;
  }

  .content {
    white-space: pre;
  }

  .mode {
    font-size: 1rem;
    text-align: center;
    color: white;
    padding: 2px 4px;
    text-transform: uppercase;
  }
  
  .mode-input {
    background-color: teal;
  }

  .mode-output {
    background-color: tomato;
  }

  button {
    border: none;
    padding: 0 2px;
    font-size: 1rem;
    font-weight: bold;
    cursor: pointer;
    background-color: rgba(255, 99, 71, 0.3);
  }

  .copied {
    color: white;
    background-color: olivedrab;
    font-weight: bold;
    text-transform: uppercase;
    padding: 2px 4px;
  }
</style>