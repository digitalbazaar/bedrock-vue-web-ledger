<template>
  <div
    class="row justify-center">
    <div
      class="column"
      style="width: 800px; max-width: 90vw;">
      <blocks
        v-if="!loading"
        :block-cache="blockCache"
        :block-id-base="blockIdBase"
        :latest-block-height="latestBlockHeight"
        :start-block="startBlock"
        :ledger-block-service="ledgerBlockService" />
      <div
        v-if="loading && showLoading"
        class="flex flex-center">
        <q-spinner-cube size="4em" />
      </div>
    </div>
  </div>
</template>
<script>
/*!
 * Copyright (c) 2018 Digital Bazaar, Inc. All rights reserved.
 */
'use strict';

import {httpClient} from '@digitalbazaar/http-client';
import Blocks from './Blocks.vue';

const headers = {Accept: 'application/ld+json, application/json'};

export default {
  name: 'BlockExplorer',
  components: {Blocks},
  props: {
    startBlock: {
      type: Number,
      default: null
    }
  },
  data() {
    return {
      loading: true,
      showLoading: false,
      // shared map from blockId to {block,meta}
      // FIXME: use LRU cache?
      blockCache: {},
      blockIdBase: null,
      latestBlockHeight: 0,
      ledgerBlockService: null
    };
  },
  async mounted() {
    const showLoadingId = setTimeout(() => this.showLoading = true, 1000);
    const laResponse = await httpClient.get('/ledger-agents/', {headers});
    // FIXME: Don't assume there is only one ledger agent
    const ledgerAgent = laResponse.data.ledgerAgent[0];
    const bsResponse =
      await httpClient.get(ledgerAgent.service.ledgerBlockService);
    const genesisBlock = bsResponse.data.genesis;
    this.blockCache[genesisBlock.block.id] = genesisBlock;

    const latestBlock = bsResponse.data.latest;
    this.blockCache[latestBlock.block.id] = latestBlock;

    this.blockIdBase = genesisBlock.block.id;
    this.blockIdBase =
      this.blockIdBase.substring(0, this.blockIdBase.length - 1);

    this.latestBlockHeight = latestBlock.block.blockHeight + 1;
    this.ledgerBlockService = ledgerAgent.service.ledgerBlockService;
    this.startBlock = this.startBlock === null ?
      this.latestBlockHeight : this.startBlock;
    clearTimeout(showLoadingId);
    this.loading = false;
  }
};
</script>
<style>
</style>
