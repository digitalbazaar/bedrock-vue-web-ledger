<template>
  <q-page class="row justify-center" padding>
    <div class="column" style="width: 800px; max-width: 90vw;">
      <h3 class="text-center">Block Explorer</h3>
      <blocks
        v-if="!loading"
        :blockCache="blockCache"
        :blockIdBase="blockIdBase"
        :latestBlockHeight="latestBlockHeight"
        :startBlock="startBlock"
        :ledgerBlockService="ledgerBlockService" />
      <div v-if="loading && showLoading" class="flex flex-center">
        <q-spinner-cube size="4em" />
      </div>
    </div>
  </q-page>
</template>
<script>
/*!
 * Copyright (c) 2018 Digital Bazaar, Inc. All rights reserved.
 */
'use strict';

import axios from 'axios';
import Blocks from './Blocks.vue';

const headers = {Accept: 'application/ld+json, application/json'};

export default {
  name: 'BlockExplorer',
  components: {Blocks},
  async mounted() {
    const showLoadingId = setTimeout(() => this.showLoading = true, 1000);
    const laResponse = await axios.get('/ledger-agents/', {headers});
    // FIXME: Don't assume there is only one ledger agent
    const ledgerAgent = laResponse.data.ledgerAgent[0];
    const bsResponse =
      await axios.get(ledgerAgent.service.ledgerBlockService);
    this.blockIdBase = bsResponse.data.genesis.block.id;
    this.blockIdBase = this.blockIdBase.substring(0, this.blockIdBase.length - 1);

    const latestBlock = bsResponse.data.latest;
    this.blockCache[latestBlock.block.id] = latestBlock;
    this.latestBlockHeight = latestBlock.block.blockHeight + 1;

    //const blocks = [bsResponse.data.genesis];
    //for(let i = 1; i <= this.latestBlockHeight; i++) {
    //  blocks.push({block: {id: blockIdBase + i}});
    //}
    //this.blocks = blocks;

    this.ledgerBlockService = ledgerAgent.service.ledgerBlockService;
    this.startBlock = this.latestBlockHeight;
    clearTimeout(showLoadingId);
    this.loading = false;
  },
  data() {
    return {
      loading: true,
      showLoading: false,
      blockCache: {},
      blockIdBase: null,
      latestBlockHeight: 0,
      startBlock: 0,
      ledgerBlockService: null
    };
  }
};
</script>
<style>
</style>
