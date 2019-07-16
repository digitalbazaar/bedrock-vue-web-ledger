<template>
  <div>
    <q-card>
      <q-card-title align="center">
        <q-field :orientation="horizontal">
          <q-input type="number"
            v-model="desiredBlock"
            prefix="Block"
            align="center"
            :suffix="'of ' + this.blocks.length"
            @blur="setBlock(desiredBlock)"
            @keyup.enter="setBlock(desiredBlock)"
          />
        </q-field>
      </q-card-title>
      <q-card-separator />
      <q-card-main>
        <tree-view :data="this.blocks[this.currentBlock - 1]"
          :options="{maxDepth: 2}"></tree-view>
      </q-card-main>

      <q-card-separator />
        <q-card-actions align="between">
          <q-btn flat color="primary" label="Previous"
            :disable="this.currentBlock <= 1" @click="previousBlock()"/>
          <q-btn class="pull-right" flat color="primary" label="Next"
            :disable="this.currentBlock >= this.blocks.length"
            @click="nextBlock()"/>
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
      desiredBlock: 1,
      currentBlock: 1
    };
  },
  props: {
    blocks: {
      type: Array,
      required: true
    },
    ledgerBlockService: {
      type: String,
      required: true
    }
  },
  watch: {
    currentBlock() {
      //console.log("CALC CBD", this.currentBlock);
      this.getCurrentBlock();
    }
  },
  methods: {
    async getCurrentBlock() {
      const fetchBlock = this.currentBlock - 1;
      // console.log("GCB", this.blocks, fetchBlock,
      //   this.ledgerBlockService, this.blocks[fetchBlock].id);
      const block = await axios.get(this.ledgerBlockService, {
        params: {
          id: this.blocks[fetchBlock].block.id
      }});
      this.blocks[fetchBlock].block = block.data.block;
      this.blocks[fetchBlock].meta = block.data.meta;
      this.currentBlockData = this.blocks[fetchBlock];
    },
    setBlock(value) {
      // console.log("SET BLOCK value", value);
      if(value > this.blocks.length) {
        this.currentBlock = this.blocks.length;
      } else if(value < 1) {
        this.currentBlock = 1;
      } else {
        this.currentBlock = value;
      }
      // console.log("SET BLOCK set", this.currentBlock);
      this.desiredBlock = this.currentBlock;
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
