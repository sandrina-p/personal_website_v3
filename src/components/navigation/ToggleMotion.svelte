<script>
  import { onMount } from 'svelte';
  import { sendTrack } from '../../utils/analytics';
  import { updateMotion } from '../../stores/motion.js';

  let isReduced = false;

  onMount(() => {
    // jsMotionReduced || jsMotionDefault
    const isDefaultReduced = document.body.classList.contains('jsMotionReduced'); // done at template.html
    setReduced(isDefaultReduced);
  });

  function setReduced (bool) {
    isReduced = bool;
    updateMotion({ isReduced })
  }

  function toggleMotion() {
    const isNowReduced = !isReduced;

    // Don't use .toggle() to avoid possible desync when clicking toooo fast.
    const actionReduced = isNowReduced ? 'add' : 'remove';
    const actionDefault = isNowReduced ? 'remove' : 'add';
    // https://github.com/sveltejs/svelte/issues/3105
    document.body.classList[actionReduced]('jsMotionReduced');
    document.body.classList[actionDefault]('jsMotionDefault');
    document.body.classList.remove('jsMotionInitial');

    setReduced(isNowReduced);
    sendTrack('reduced_motion_toggle', isNowReduced);
  }

  export let klass = '';
</script>

<style lang="postcss">
  @define-mixin motionDefault { :global(.jsMotionDefault) & { @mixin-content; } }
  @define-mixin motionReduced { :global(.jsMotionReduced) & { @mixin-content; } }

  $size: 1.8rem;

  .toggleMotion {
    position: relative;
    background: transparent;
    border: none;
    padding: 0;
    color: inherit;
    cursor: pointer;
    white-space: nowrap;
    font-size: $font-M;
    font-family: inherit;
    font-weight: 300;
    padding: calc($spacer-S + $spacer-XS) $spacer-S;
    white-space: nowrap;
    display: flex;
    align-items: center;

    &:hover {
      color: var(--primary_1);
      .btnDecor {
        border-color: var(--primary_1);
      }
    }

    &:focus {
      outline: none;
    }

    &[aria-pressed="true"] {
      .btnDecor::before {
        background-color: var(--primary_1);
        --ty: 50%;
      }
    }
  }

  .btnDecor {
    position: relative;
    display: block;
    width: calc($size*1.5);
    height: $size;
    border: 1px solid var(--text_0);
    border-radius: $size;
    margin-left: 0.8rem;
    box-sizing: content-box;
    
    &::before {
      content: '';
      position: absolute;
      display: block;
      width: $size;
      height: $size;
      border-radius: 50%;
      box-sizing: border-box;
      border: 2px solid var(--bg_1);
      background-color: var(--text_1);
      transform: translateX(var(--ty, 0));
      transition: background-color 250ms, transform 250ms cubic-bezier(0.0, 0.0, 0.2, 1); /* irony */
    }
  }
</style>

<button class="toggleMotion {klass}" on:click={toggleMotion} aria-pressed={isReduced}>
  Reduced motion
  <span class="btnDecor"></span>
</button>

