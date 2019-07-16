<template>
  <q-page class="row justify-center" padding>
    <div class="column" style="width: 800px; max-width: 90vw;">

      <h3 class="text-center">Block Explorer</h3>

      <blocks :blocks="ledgerBlocks" :ledgerBlockService="ledgerBlockService" />

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
    const laResponse = await axios.get('/ledger-agents/', {headers});
    // FIXME: Don't assume there is only one ledger agent
    const ledgerAgent = laResponse.data.ledgerAgent[0];
    const bsResponse =
      await axios.get(ledgerAgent.service.ledgerBlockService);
    const latestBlockHeight = bsResponse.data.latest.block.blockHeight;
    let blockIdPattern = bsResponse.data.genesis.block.id;
    blockIdPattern = blockIdPattern.substring(0, blockIdPattern.length - 1);

    const blocks = [bsResponse.data.genesis];
    for(let i = 1; i <= latestBlockHeight; i++) {
      blocks.push({block: {id: blockIdPattern + i}});
    }

    this.ledgerBlockService = ledgerAgent.service.ledgerBlockService;
    this.ledgerBlocks = blocks;
  },
  data() {
    return {
      ledgerBlockService: null,
      ledgerBlocks: []
    };
  },
  validations: {
  },
  watch: {
  },
  methods: {
  }
};
</script>
<style>
</style>
