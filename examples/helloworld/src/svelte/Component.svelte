<script>
    // imports
    import Button from './Button.svelte';
    import interpolate from './interpolate.js';
    import {setup} from './environment.js';
    import {slide} from 'svelte/transition';

    setup({
        exampleServerSideFunction: function (value) {
            return value + ' (locally)';
        }
    });

    // prop declarations
    export let greeting = 'hello';
    export let noun = 'world';
    export let template = '${greeting}, ${noun}';
    export let language = 'English';
    export let index = 0;

    // callback handlers
    const success = (valueInside, userObj) => {
        const obj = userObj();
        template += ' ' + valueInside + obj.config.exclamation + ' #' + (obj.index + 1);
    };

    // execute serverSideFunction (immediately)
    google.script.run
        .withUserObject( () => ({
            config: {
                exclamation: '!',
                question: '?'
            },
            index
        }) )
        .withSuccessHandler(success)
        .exampleServerSideFunction('from inside appscripts');


    let html, plaintext;
    /*
        Two reactive statements ensure that the information is updated when template changes
        (as it does when callback is used)
     */
    $: html = interpolate(template, {
        greeting: `<span class="greeting">${greeting}</span>`,
        noun: `<span class="noun">${noun}</span>`
    });

    $: plaintext = interpolate(template, {greeting, noun});

</script>

<div transition:slide="{{delay: index * 500}}">
<h2>{language}</h2>
<div class="panel">
    <h4>Plain-text Output:</h4>
        <p>{@html html}</p>
    <h4>Html Output:</h4>
        <p class="html">{ html }</p>
</div>
<p>
    <!--
        Use Button component, so that this component bubbles up `makeTitle` event,
        and sends it language and title info
    -->
    <Button on:makeTitle title={plaintext} {language} />
</p>
</div>

<style>
    div.panel {
        background-color: #eee;
        border: 1px solid #333;
        padding: 0 10px;
    }
    p :global(span.noun) {
        color: purple;
    }
    p :global(span.greeting) {
        color: blue;
    }
</style>
