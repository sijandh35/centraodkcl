<!--
Copyright 2017 ODK Central Developers
See the NOTICE file at the top-level directory of this distribution and at
https://github.com/getodk/central-frontend/blob/master/NOTICE.

This file is part of ODK Central. It is subject to the license terms in
the LICENSE file found in the top-level directory of this distribution and at
https://www.apache.org/licenses/LICENSE-2.0. No part of ODK Central,
including this file, may be copied, modified, propagated, or distributed
except according to the terms contained in the LICENSE file.
-->
<template>
  <div v-show="state" class="loading">{{ $t('common.loading') }}</div>
</template>

<script>
export default {
  name: 'Loading',
  props: {
    state: {
      type: Boolean,
      default: false
    }
  }
};
</script>

<style lang="scss">
.loading {
  color: #999;
  margin-bottom: 10px;
  text-align: center;
}
</style>
