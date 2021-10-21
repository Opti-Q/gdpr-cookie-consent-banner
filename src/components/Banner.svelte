<script>
  import Cookie from 'cookie-universal'
  import { validate } from '../util'
  import { fade } from 'svelte/transition'
  import { onMount, createEventDispatcher } from 'svelte'

  const dispatch = createEventDispatcher()
  const cookies = Cookie()

  export let cookieName = null
  export let showEditIcon = true

  let shown = false
  let settingsShown = false

  export let heading = 'GDPR Notice'
  export let description =
    'We use cookies to offer a better browsing experience, analyze site traffic, personalize content, and serve targeted advertisements. Please review our privacy policy & cookies information page. By clicking accept, you consent to our privacy policy & use of cookies.'

  export let categories = {
    analytics: function () {},
    tracking: function () {},
    marketing: function () {},
    necessary: function () {}
  }

  export let cookieConfig = {}

  export let choices = {}
  const choicesDefaults = {
    necessary: {
      label: 'Necessary cookies',
      description: "Used for cookie control. Can't be turned off.",
      value: true
    },
    tracking: {
      label: 'Tracking cookies',
      description: 'Used for advertising purposes.',
      value: false
    },
    analytics: {
      label: 'Analytics cookies',
      description:
        'Used to control Google Analytics, a 3rd party tool offered by Google to track user behavior.',
      value: false
    },
    marketing: {
      label: 'Marketing cookies',
      description: 'Used for marketing data.',
      value: false
    }
  }

  $: choicesMerged = Object.assign({}, choicesDefaults, choices)

  $: choicesArr = Object.values(choicesMerged).map((item, index) => {
    return Object.assign(
      {},
      item,
      { id: Object.keys(choicesMerged)[index] }
    )
  })
  
  $: cookieChoices = choicesArr.reduce((result, item, index, array) => {
    result[item.id] = item.value ? item.value : false
    return result
  }, {})

  export let acceptLabel = 'Accept all'
  export let settingsLabel = 'Change settings'
  export let closeLabel = 'Save'
  export let selectAllLabel = 'Select all'

  export function show () {
    shown = true
  }

  onMount(() => {
    if (!cookieName) {
      throw new Error('You must set gdpr cookie name')
    }

    const cookie = cookies.get(cookieName)
    if (cookie && chosenMatchesChoice(cookie)) {
      execute(cookie.choices)
    } else {
      removeCookie()
      show()
    }
  })

  function setCookie (choices) {
    const expires = new Date()
    expires.setDate(expires.getDate() + 365)

    const options = Object.assign({}, cookieConfig, { expires })
    cookies.set(cookieName, { choices }, options)
  }

  function removeCookie () {
    const { path } = cookieConfig
    cookies.remove(cookieName, Object.assign({}, path ? { path } : {}))
  }

  function chosenMatchesChoice (cookie) {
    return validate(cookieChoices, cookie)
  }

  function execute (chosen) {
    const types = Object.keys(cookieChoices)

    types.forEach(t => {
      const agreed = chosen[t]
      if (choicesMerged[t]) {
        choicesMerged[t].value = agreed
      }
      if (agreed) {
        categories[t] && categories[t]()
        dispatch(`${t}`)
      }
    })
    shown = false
  }

  function acceptAll(){
    for(let choice of choicesArr){
      if(Object.hasOwnProperty.call(choicesMerged, choice.id) && choicesMerged[choice.id]){
        choicesMerged[choice.id].value = true;
      }
    }
    var choicesA = Object.values(choicesMerged).map((item, index) => {
      return Object.assign(
        {},
        item,
        { id: Object.keys(choicesMerged)[index] }
      )});
    var choices = choicesA.reduce((result, item, index, array) => {
      result[item.id] = item.value ? item.value : false
      return result
    }, {});

    choose(choices);
  }

  function choose (choices) {
    setCookie(choices ?? cookieChoices)
    execute(choices ?? cookieChoices)
  }
</script>

{#if showEditIcon}
  <button
    class="cookieConsentToggle"
    on:click={show}
    transition:fade>
    
    <i class="fa fa-user-secret fa-lg" aria-hidden="true"></i>
  </button>
{/if}

{#if shown}
<div class="cookieConsentWrapper" transition:fade>
  <div class="cookieConsent">
    <div class="cookieConsent__Left">
      <div class="cookieConsent__Content">
        <p class="cookieConsent__Title">{heading}</p>
        <p class="cookieConsent__Description">
          {@html description}
        </p>
      </div>
    </div>
    <div class="cookieConsent__Right">
      <button
        type="button"
        class="cookieConsent__Button"
        on:click={() => { settingsShown = true } }>
        {settingsLabel}
      </button>
      <button type="submit" class="cookieConsent__Button cookieConsent__Button--Primary" on:click={acceptAll}>
        {acceptLabel}
      </button>
    </div>
  </div>
</div>
{/if}

{#if settingsShown}
<div class="cookieConsentOperations" transition:fade>
  <div class="cookieConsentOperations__List">
    {#each choicesArr as choice}
      {#if Object.hasOwnProperty.call(choicesMerged, choice.id) && choicesMerged[choice.id]}
        <div
          class="cookieConsentOperations__Item"
          class:disabled={choice.id === 'necessary'}>
          <input
            type="checkbox"
            id={`gdpr-check-${choice.id}`}
            bind:checked={choicesMerged[choice.id].value}
            disabled={choice.id === 'necessary'} />
          <label for={`gdpr-check-${choice.id}`}>{choice.label}</label>
          <span class="cookieConsentOperations__ItemLabel">
            {choice.description}
          </span>
        </div>
      {/if}
    {/each}
    <div class="cookieConsent__Center">
      <button
        type="submit"
        class="cookieConsent__Button cookieConsent__Button--Close"
        on:click={() =>
          { 
            settingsShown = false;
            choose(); 
          }
        }>
        {closeLabel}
      </button>
      <button
        type="submit"
        class="cookieConsent__Button cookieConsent__Button--Primary"
        on:click={() => {
          acceptAll();
          settingsShown = false 
          } }>
        {selectAllLabel}
      </button>
    </div>
  </div>
</div>
{/if}
