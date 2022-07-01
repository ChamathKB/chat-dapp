<script>
    import login from './Login.svelte';
    import ChatMessge from './ChatMessge.svelte';
    import { onMount } from 'svelte';
    import { username, user } from './user';
    import debounce from 'loadash.debounce';

    import GUN from 'gun';
    import { dataset_dev } from 'svelte/internal';
    const db = GUN();

    let NewMessage;
    let message = [];

    let scrollBottom;
    let lastScrollTop;
    let canAutoScroll = true;
    let unreadMessage = false;

    function autoScroll() {
        setTimeout(() => scrollBottom?.scrollIntoView({ behaviour: 'auto' }), 50);
        unreadMessage = false;
    }

    function watchScroll(e) {
        canAutoScroll = (e.target.scrollTop || Infinity) > lastScrollTop;
        lastScrollTop = e.target.scrollTop;
    }

    $: debounceWatchScroll = debounce(watchScroll, 1000);

    onMount(() => {
        var match = {
      // lexical queries are kind of like a limited RegEx or Glob.
      '.': {
        // property selector
        '>': new Date(+new Date() - 1 * 1000 * 60 * 60 * 3).toISOString(), // find any indexed property larger ~3 hours ago
      },
      '-': 1, // filter in reverse            
        };

        // get message
        db.get('chat')
          .map(match)
          .once(async (data, id) => {
            if (data) {
                const key = '#foobar?../@#$' // key for end-to-end encryption (not safe)

                var message = {
                    who: await db.user(data).get('alias'), // a user might lie who they are! So let the user system detect whose data it is.
                    what: (await SEA.decrypt(data.what, key)) + '', // force decrypt as text.
                    when: GUN.state.is(data, 'what'), // get the internal timestamp for the what property. 
                };
                if (message.what) {
                    messages = [...messages.slice(-100), message].sort((a, b) => a.when - b.when);
                    if (canAutoScroll) {
                        autoScroll();
                    } else {
                        unreadMessages = true;
                    }                
                }
            }
        });
    });

    async function sendMessage() {
        const secret = await SEA.encrypt(newMessage, '#foo');
        const message = user.get('all').set({ what: secret });
        const index = new Date().toISOString();
        db.get('chat').get(index).put(message);
        newMessage = '';
        canAutoScroll = true;
        autoScroll();
  }
</script>

<div class="container">
    {#if $username}
      <main on:scroll={debouncedWatchScroll}>
        {#each messages as message (message.when)}
          <ChatMessage {message} sender={$username} />
        {/each}
        <div class="dummy" bind:this={scrollBottom} />
      </main>
      <form on:submit|preventDefault={sendMessage}>
        <input type="text" placeholder="Type a message..." bind:value={newMessage} maxlength="100" />
        <button type="submit" disabled={!newMessage}>💥</button>
      </form>
      {#if !canAutoScroll}
      <div class="scroll-button">
        <button on:click={autoScroll} class:red={unreadMessages}>
          {#if unreadMessages}
            💬
          {/if}
          👇
        </button>
      </div>
     {/if}
    {:else}
      <main>
        <Login />
      </main>
    {/if}
</div>