<style>
    .messages {
        overflow-y: scroll;
        min-height: 300px;
    }
    .messages * {
        overflow-anchor: none;
    }
    .messages .anchor {
        overflow-anchor: auto;
        height: 1px;
    }
</style>

<script>
    import * as Game from '../stores/game.js';
    import { makeMessage } from '../factory.js';
    import { roll } from '../util.js';

    export let filter = () => true;

    let session, messages = [];
    let from = 'anon';
    if (process.browser) {
        Game.store.subscribe(state => {
            messages = state.messages;
            session = state.session;
        });
    }

    async function handleSendMessage (e) {
        if (e.key !== 'Enter') return;

        e.preventDefault();

        let message = e.target.value;
        let to = 'all';

        if (message.match(/^\/(gm?)?r(oll)? /)) {
            if (message.startsWith('/g')) to = 'gm';

            const v = message.replace(/^\/(gm?)?r(oll)? /, '');
            const { results, total } = roll(v);
            message = `rolled (${v.trim()}) to ${to} ${results.join(' + ')} = ${total}`;
        }

        await Game.insertMessage(makeMessage(session.id, { from, to, message }));

        e.target.value = '';
    }
</script>

<div class="col hide-overflow {$$props.class}">
    <div class="messages col grow">
        {#each messages as message}
            {#if filter(message)}
                <div>
                    <span>{message.from}</span> - <span>{message.message}</span>
                </div>
            {/if}
        {/each}
        <div class="anchor">&nbsp;</div>
    </div>
    <div>
        <label class="row-centered">
            From:&nbsp;
            <input type="text" bind:value={from} class="grow" />
        </label>
        <textarea
            style="width: 100%;"
            on:keydown={handleSendMessage}
        />
    </div>
</div>
