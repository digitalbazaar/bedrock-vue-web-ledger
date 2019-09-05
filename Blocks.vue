<template>
  <div>
    <q-card>
      <q-card-section>
        <div class="row items-center no-wrap">
          <div class="col">
            <div class="text-h5">Block <strong>{{currentBlock}}</strong> of <strong>{{latestBlockHeight}}</strong></div>
          </div>
          <div class="col-auto">
            <q-btn flat icon="fas fa-search" @click="showSearch=!showSearch"/>
          </div>
        </div>
        <div v-if="showSearch">
          <q-input outlined
            type="number" min="1" :max="latestBlockHeight"
            v-model="desiredBlock"
            label="Load Block"
            @change="setBlock(desiredBlock)"
            @blur="setBlock(desiredBlock)"
            @keyup.enter="setBlock(desiredBlock)"
            />
        </div>
      </q-card-section>

      <q-separator />

      <q-card-section>
        <tree-view :data="currentBlockData"
          :options="{maxDepth: 2}"></tree-view>
      </q-card-section>

      <q-separator />

      <q-card-actions align="center">
        <q-pagination
          v-model="currentBlock"
          min="1"
          :max="latestBlockHeight"
          :input="true"
          @input="setBlock(currentBlock)"
          >
        </q-pagination>
      </q-card-actions>
    </q-card>
  </div>
</template>
<script>
/*!
 * Copyright (c) 2018 Digital Bazaar, Inc. All rights reserved.
 */
'use strict';

import axios from 'axios';

export default {
  name: 'Blocks',
  components: {},
  data() {
    return {
      // map from blockId to {block,meta}
      // FIXME: use LRU cache or shared prop cache?
      blockCache: {},
      desiredBlock: this.startBlock,
      currentBlock: null,
      currentBlockData: null,
      showSearch: false
    };
  },
  props: {
    blockCache: {
      type: Object,
      required: false
    },
    blockIdBase: {
      type: String,
      required: true
    },
    latestBlockHeight: {
      type: Number,
      required: true
    },
    startBlock: {
      type: Number,
      default: () => this.latestBlockHeight,
      required: false
    },
    ledgerBlockService: {
      type: String,
      required: true
    }
  },
  mounted() {
    this.setBlock(this.startBlock);
  },
  methods: {
    makeBlockId(blockHeight) {
      return this.blockIdBase + blockHeight;
    },
    async getCurrentBlock() {
      const fetchBlockId = this.makeBlockId(this.currentBlock - 1)
      if(!(fetchBlockId in this.blockCache)) {
        const block = await axios.get(this.ledgerBlockService, {
          params: {
            id: fetchBlockId
          }
        });
        this.blockCache[fetchBlockId] = {
          block: block.data.block,
          meta: block.data.meta
        };
      }
      this.currentBlockData = this.blockCache[fetchBlockId];
    },
    clampBlock(value) {
      return Math.max(1, Math.min(value, this.latestBlockHeight));
    },
    setBlock(value) {
      // clamp between 1 and total blocks
      this.desiredBlock = this.clampBlock(value);
      this.currentBlock = this.desiredBlock;
      this.getCurrentBlock();
    },
    nextBlock() {
      this.setBlock(this.currentBlock + 1);
    },
    previousBlock() {
      this.setBlock(this.currentBlock - 1);
    }
  }
};
</script>
<style>
</style>
