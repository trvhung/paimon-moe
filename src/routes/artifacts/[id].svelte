<script context="module">
  import data from '../../data/artifacts/en.json';
  import { builds } from '../../data/build';

  function getCharacter(artifactId) {
    const collection2 = {};
    const collection4 = {};
    const collectionPiece = {
      sands: {},
      goblet: {},
      circlet: {},
    };

    const chars = Object.entries(builds);
    for (const [charId, char] of chars) {
      const roles = Object.entries(char.roles);
      for (const [roleName, role] of roles) {
        if (!role.recommended) continue;

        let found2 = false;
        let found4 = false;
        for (const artifact of role.artifacts) {
          if (
            artifact.find((e) => {
              if (e === '+18%_atk_set') {
                return [
                  'gladiators_finale',
                  'shimenawas_reminiscence',
                  'vermillion_hereafter',
                  'echoes_of_an_offering',
                ].includes(artifactId);
              } else if (e === '+20%_energy_recharge') {
                return ['emblem_of_severed_fate', 'the_exile', 'scholar'].includes(artifactId);
              } else if (e === '+25%_physical_dmg') {
                return ['bloodstained_chivalry', 'pale_flame'].includes(artifactId);
              }

              return e === artifactId;
            })
          ) {
            if (artifact.length === 1) found4 = true;
            else found2 = true;

            for (const piece of ['sands', 'goblet', 'circlet']) {
              for (const stat of role.mainStats[piece]) {
                let statName = stat;
                if (stat === 'DMG') statName = 'Crit DMG';
                if (collectionPiece[piece][statName] === undefined) collectionPiece[piece][statName] = {};
                collectionPiece[piece][statName][charId] = true;
              }
            }
          }
        }

        if (found2) {
          if (collection2[charId] === undefined) {
            collection2[charId] = {
              id: charId,
              roles: [],
            };
          }

          collection2[charId].roles.push(roleName);
        }

        if (found4) {
          if (collection4[charId] === undefined) {
            collection4[charId] = {
              id: charId,
              roles: [],
            };
          }

          collection4[charId].roles.push(roleName);
        }
      }
    }

    return {
      two: Object.values(collection2).sort((a, b) => a.id.localeCompare(b.id)),
      four: Object.values(collection4).sort((a, b) => a.id.localeCompare(b.id)),
      pieces: collectionPiece,
    };
  }

  export async function load({ params }) {
    const { id } = params;
    const artifact = data[id];
    const recommendedCharacter = getCharacter(id);

    return { props: { id, artifact, recommendedCharacter } };
  }
</script>

<script>
  import { getContext, onMount } from 'svelte';
  import { mdiChevronRight, mdiCircle, mdiStar } from '@mdi/js';
  import { locale, t } from 'svelte-i18n';
  import Icon from '../../components/Icon.svelte';
  import { domains } from '../../data/domain.js';
  import { characters } from '../../data/characters';
  import PieceModal from './_pieceModal.svelte';

  const { open: openModal } = getContext('simple-modal');

  const rarityList = {
    1: 'text-white',
    2: 'text-green-400',
    3: 'text-primary',
    4: 'text-rare-from',
    5: 'text-legendary-from',
  };

  const pieces = ['flower', 'plume', 'sands', 'goblet', 'circlet'];

  export let id;
  export let artifact;
  export let recommendedCharacter;
  // console.log(recommendedCharacter.pieces);
  let images = [];

  async function changeLocale(locale) {
    const _data = await import(`../../data/artifacts/${locale}.json`);
    artifact = _data.default[id];
  }

  function openPieceModal(i) {
    if (i < 2) return;

    const piece = pieces[i];
    const current = Object.entries(recommendedCharacter.pieces[piece]).map((e) => [
      e[0],
      Object.keys(e[1]).sort((a, b) => a.localeCompare(b)),
    ]);

    openModal(
      PieceModal,
      {
        pieces: current,
        name: piece,
      },
      {
        closeButton: false,
        styleWindow: { background: '#25294A', width: '816px' },
      },
    );
  }

  onMount(async () => {
    locale.subscribe((val) => {
      changeLocale(val);
    });

    const maxPiece = artifact.setPiece[artifact.setPiece.length - 1];
    if (maxPiece === 1) {
      images = [`${id}_circlet.png`];
    } else {
      images = ['flower', 'plume', 'sands', 'goblet', 'circlet'].map((e) => `${id}_${e}.png`);
    }
  });

  $: rarity = artifact.rarity[artifact.rarity.length - 1];
</script>

<svelte:head>
  <title>Genshin Impact {artifact.name} - HoyoHub</title>
</svelte:head>
<div class="lg:ml-64 pt-20 lg:pt-8 max-w-screen-xl md:pl-4">
  <div class="flex flex-col items-start">
    <div class="px-4">
      <h1 class="font-display font-black text-4xl md:text-5xl text-white">{artifact.name}</h1>
      <div class="{rarityList[rarity]} text-2xl flex items-center z-0 -mt-2 md:-mt-4">
        {#each [...new Array(rarity)] as _}
          <Icon path={mdiStar} />
        {/each}
        <Icon path={mdiCircle} size={0.4} className="mx-2 mt-1" color="white" />
        <p class="text-base text-white font-semibold mt-1">{$t(`artifact.artifact`)}</p>
      </div>
    </div>
    <p class="pt-1 pl-4 text-gray-400">※ {$t('artifact.pieceInfo')}</p>
    <div class="px-4 pt-4 flex flex-wrap space-x-2 justify-center">
      {#each images as image, i}
        <div class={i > 1 ? 'hover:bg-background rounded-xl cursor-pointer' : ''} on:click={() => openPieceModal(i)}>
          <img src="/images/artifacts/{image}" alt={id} class="w-24 h-24" />
        </div>
      {/each}
    </div>
    <div class="px-4 pt-4 flex flex-col space-y-2 max-w-xl w-full">
      {#each artifact.bonuses as bonus, i}
        <div class="bg-background rounded-xl p-4 text-white">
          <p class="font-bold mb-1 text-primary">{$t('artifact.setPiece', { values: { piece: (i + 1) * 2 } })}</p>
          <p>{bonus}</p>
        </div>
      {/each}
      {#if artifact.domain}
        <a
          href="/domains/{artifact.domain}"
          class="flex justify-end items-center self-end lg:self-start text-white mt-4 
    bg-item rounded-xl py-2 px-4 hover:bg-background transition-colors duration-100"
        >
          {$t('artifact.domain')}: {domains[artifact.domain].name}
          <Icon path={mdiChevronRight} />
        </a>
      {/if}
    </div>
    {#if recommendedCharacter.two.length > 0}
      <div class="mt-6 max-w-screen-lg">
        <h3 class="font-display font-bold text-2xl text-white px-4">
          {$t('artifact.recommendedCharacter', { values: { piece: 2 } })}
        </h3>
        <div class="flex flex-wrap px-4 -mx-1 -mb-2">
          {#each recommendedCharacter.two as char}
            <a
              class="flex items-center bg-background hover:bg-item rounded-xl px-2 py-1 mb-2 mx-1"
              href="/characters/{char.id}"
            >
              <img
                src="/images/characters/{char.id}.png"
                alt={characters[char.id].name}
                class="w-12 h-12 mr-2 rounded-full"
              />
              <div class="-mx-1">
                {#each char.roles as role}
                  <p class="text-white mx-1">{role}</p>
                {/each}
              </div>
            </a>
          {/each}
        </div>
      </div>
    {/if}
    {#if recommendedCharacter.four.length > 0}
      <div class="mt-4 max-w-screen-lg">
        <h3 class="font-display font-bold text-2xl text-white px-4">
          {$t('artifact.recommendedCharacter', { values: { piece: 4 } })}
        </h3>
        <div class="flex flex-wrap px-4 -mx-1 -mb-2">
          {#each recommendedCharacter.four as char}
            <a
              class="flex items-center bg-background hover:bg-item rounded-xl px-2 py-1 mb-2 mx-1"
              href="/characters/{char.id}"
            >
              <img
                src="/images/characters/{char.id}.png"
                alt={characters[char.id].name}
                class="w-12 h-12 mr-2 rounded-full"
              />
              <div class="-mx-1">
                {#each char.roles as role}
                  <p class="text-white mx-1">{role}</p>
                {/each}
              </div>
            </a>
          {/each}
        </div>
      </div>
    {/if}
  </div>
</div>
