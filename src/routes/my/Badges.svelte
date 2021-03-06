<script>
  export let router;

  import { onMount } from 'svelte';
  import { getClient } from '@urql/svelte';

  import { ActionHeader, LinkButton } from '../../elements';
  import StackedLayout from '../../elements/layouts/StackedLayout.svelte';
  import Nav from '../../components/nav/interiorNav/Top.svelte';
  import Warning from '../../components/notifications/Warning.svelte';
  import ClaimTicketForm from '../../components/my/ClaimTicketForm.svelte';
  import memberApi from '../../dataSources/api.that.tech/members.js';
  import { tagEvent } from '../../utilities/gtag';

  import {
    isAuthenticated,
    user,
    thatProfile,
    refreshMe,
  } from '../../utilities/security.js';

  const { claimTicket } = memberApi(getClient());

  let awardedBadge;
  let failedClaim = false;

  let awardedBadges = [];
  $: if ($thatProfile.earnedMeritBadges) {
    awardedBadges = [...$thatProfile.earnedMeritBadges];
  }

  async function handleClaimTicket({
    detail: { values, setSubmitting, resetForm },
  }) {
    failedClaim = false;

    const { ticketReference } = values;
    const badgeEarned = await claimTicket(ticketReference);

    if (badgeEarned) {
      tagEvent('badgeClaimed', 'account', badgeEarned.id);
      awardedBadge = badgeEarned;

      await refreshMe();
      resetForm();
    } else {
      failedClaim = true;
    }

    tagEvent('claim_badge', 'account', $user.sub);
    setSubmitting(false);
  }
</script>

<svelte:head>
  <title>Merit Badges * THAT.us</title>
</svelte:head>

<StackedLayout>
  <div slot="header">
    <Nav />
    <ActionHeader title="Your Merit Badges">
      <LinkButton href="/sessions" text="Return to Schedule" />
    </ActionHeader>
  </div>

  <div slot="body">
    {#if awardedBadges.length > 0}
      <div>
        <h2
          class="text-3xl leading-9 font-extrabold tracking-tight text-gray-900
          sm:text-4xl sm:leading-10"
        >
          Awarded Badges
        </h2>

        <div class="mt-12">
          {#each awardedBadges as badge (badge.id)}
            <div class="flex space-x-3">
              <div class="flex flex-col items-center">
                <img class="h-56 w-56" src="{badge.image}" alt="{badge.name}" />
                <h2
                  class="text-xl leading-9 font-bold tracking-tight
                  text-gray-500 sm:text-2xl sm:leading-10"
                >
                  {badge.name}
                </h2>
              </div>
            </div>
          {/each}
        </div>
      </div>
    {/if}

    <div class="mt-12 border-t">
      <div class="pt-6">
        <ClaimTicketForm handleSubmit="{handleClaimTicket}" />
      </div>
    </div>
  </div>

  <div slot="footer"></div>
</StackedLayout>

{#if failedClaim}
  <Warning
    message="We were unable to claim that ticket number. Please re-check and try
    again."
  />
{/if}
