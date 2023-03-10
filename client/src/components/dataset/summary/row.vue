<!--
Copyright 2022 ODK Central Developers
See the NOTICE file at the top-level directory of this distribution and at
https://github.com/getodk/central-frontend/blob/master/NOTICE.

This file is part of ODK Central. It is subject to the license terms in
the LICENSE file found in the top-level directory of this distribution and at
https://www.apache.org/licenses/LICENSE-2.0. No part of ODK Central,
including this file, may be copied, modified, propagated, or distributed
except according to the terms contained in the LICENSE file.
-->
<template>
  <div class="dataset-summary-row">
    <!-- eslint-disable-next-line vuejs-accessibility/click-events-have-key-events, vuejs-accessibility/interactive-supports-focus -->
    <div class="row" role="button" @click.prevent="toggleExpanded">
      <div class="col-xs-6 dataset-name-wrap">
        <div class="dataset-name text-overflow-ellipsis" :title="dataset.name">{{ dataset.name }}</div>
        <div v-if="dataset.isNew" class="dataset-new">
          <span class="icon-plus-circle"></span>
          {{ $t('new') }}
        </div>
      </div>
      <div class="col-xs-6 properties-count">
        {{ $tcn('properties', dataset.properties.length, { inform: $n(inFormProperties.length, 'default') }) }}
        <!-- eslint-disable-next-line vuejs-accessibility/anchor-has-content -->
        <a href="javascript:void(0)" class="expand-button">
          <span v-if="!expanded" class="icon-caret-left"></span>
          <span v-else class="icon-caret-down"></span>
        </a>
      </div>
    </div>
    <div v-show="expanded" class="property-list">
      <span v-for="(property, index) in inFormProperties" :key="property.name">
        <span>{{ property.name }}</span>
        <span v-if="property.isNew" class="icon-plus-circle property-new" :title="$t('addedByThisDraft')"></span>
        <template v-if="index < inFormProperties.length - 1">{{ $t('common.punctuations.comma') }}<sentence-separator/></template>
      </span>
    </div>
  </div>
</template>

<script>
import SentenceSeparator from '../../sentence-separator.vue';

export default {
  name: 'DatasetSummaryRow',
  components: { SentenceSeparator },
  props: {
    dataset: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      expanded: false
    };
  },
  computed: {
    newProperties() {
      return this.dataset.properties.filter(p => p.isNew);
    },
    inFormProperties() {
      return this.dataset.properties.filter(p => p.inForm);
    }
  },
  methods: {
    toggleExpanded() {
      this.expanded = !this.expanded;
    }
  }
};

</script>

<style lang="scss">
@import '../../../assets/scss/_variables.scss';
@import '../../../assets/scss/mixins';

.dataset-summary-row {
    @include text-block;

    .text-overflow-ellipsis {
      @include text-overflow-ellipsis;
    }

    .dataset-name-wrap{
      display: flex;

      .dataset-name {
        font-weight: bold;
        font-size: 18px;
      }
      .dataset-new {
          vertical-align: super;
          color: $color-success;
          margin-left: 2px;
          min-width: 46px;
      }
    }

    .property-new {
        margin-left: 2px;
        color: $color-success;
        vertical-align: super;
    }

    .properties-count {
        line-height: 28px;
    }

    a {
        color: #666;
        font-size: 20px;
        vertical-align: middle;

        &:focus {
            background-color: inherit;
        }
    }

    .expand-button {
      margin-left: 5px;
    }

    .property-list {
      hyphens: auto;
      overflow-wrap: break-word;
    }
}

</style>

<i18n lang="json5">
{
  "en": {
    // This is shown when a dataset is new
    "new": "new!",
    // {inform} is the number of dataset properties defined by the form.
    // {count} is the total number of properties. The string will be pluralized based on {count}.
    "properties": "{inform} of {count} property | {inform} of {count} properties",
    // This is shown when mouse hovers over plus icon of new Dataset Property
    "addedByThisDraft": "Added by this Draft"
  }
}
</i18n>

<!-- Autogenerated by destructure.js -->
<i18n>
{
  "cs": {
    "new": "nov??!"
  },
  "fr": {
    "new": "nouveau !",
    "properties": "{inform} des {count} propri??t??s | {inform} des {count} propri??t??s | {inform} des {count} propri??t??s",
    "addedByThisDraft": "Ajout?? par cette ??bauche"
  }
}
</i18n>
