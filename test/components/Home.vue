<template>
  <q-page
    class="column q-col-gutter-md background"
    padding>
    <br-block-explorer fixme='FIXME'></br-block-explorer>
  </q-page>
</template>
<script>
/*!
 * Copyright (c) 2019 Digital Bazaar, Inc. All rights reserved.
 */
'use strict';

import {BrBlockExplorer} from 'bedrock-vue-web-ledger';

export default {
  name: 'Home',
  components: {BrBlockExplorer},
  data() {
    return {
      webLedgerData: {
        // FIXME
      }
    };
  }
};
</script>
<style scoped>
.full-width {
  width: 100%;
}
</style>
