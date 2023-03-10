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
  <div id="dataset-introduction" class="panel-dialog">
    <div class="panel-heading">
      <span class="panel-title h1">{{ $t('title') }}</span>
    </div>
    <div class="panel-body row">
      <div class="col-md-3">
        <p>{{ $t('body[0]') }}</p>
        <i18n-t tag="p" keypath="body[1]">
          <template #icon1><span class="step-number">(1)</span></template>
        </i18n-t>
        <i18n-t tag="p" keypath="body[2]">
          <template #icon2><span class="step-number">(2)</span></template>
          <template #icon3><span class="step-number">(3)</span></template>
        </i18n-t>
        <i18n-t tag="p" keypath="body[3]">
          <template #icon4><span class="step-number">(4)</span></template>
        </i18n-t>
        <p>
          <span>{{ $t('body[4]') }}</span>
          <sentence-separator/>
          <i18n-t keypath="getStarted.full">
            <template #clickHere>
              <doc-link to="central-datasets/">{{ $t('getStarted.clickHere') }}</doc-link>
            </template>
          </i18n-t>
        </p>
      </div>
      <div class="col-md-9 image-wrapper">
        <img src="../../../public/images/entities-intro.svg" :alt="$t('datasetsWorkflow')">
      </div>
    </div>
  </div>
</template>

<script>
import DocLink from '../doc-link.vue';
import SentenceSeparator from '../sentence-separator.vue';

export default {
  name: 'DatasetIntroduction',
  components: { DocLink, SentenceSeparator }
};
</script>

<style lang="scss">
@import '../../assets/scss/_variables.scss';

#dataset-introduction {
  &.panel-dialog {
    width: 100%;
    max-width: 770px;
    margin-bottom: 20px;

    .panel-title {
      font-size: 30px;
    }
  }

  .step-number {
    font-style: italic;
  }

  .image-wrapper img {
    width: 100%;
    max-width: 720px;
  }
}

@media (min-width: $screen-md-min) {
  #dataset-introduction {
    &.panel-dialog {
      max-width: 1045px;
    }

    .row {
      display: flex;
    }

    .image-wrapper {
      display: flex;
    }
  }
}
</style>

<i18n lang="json5">
{
  "en": {
    "title": "Welcome to the Datasets preview!",
    "body": [
      "Datasets are a new way to create and manage data attachments in ODK. They allow you to use data gathered in Submissions to be downloaded and used as part of another Form.",
      // {icon1} is an icon that displays the number 1. It refers to the first step in an image that depicts this workflow.
      "Datasets can be attached {icon1} to Forms and their data queried just like any other attached CSV data file.",
      // {icon2} and {icon3} are icons that display the numbers 2 and 3. They refer to the second and third steps in an image that depicts this workflow.
      "When a Submission related to a Dataset is approved {icon2}, the selected data is added {icon3} to the Dataset, so that the next time a client downloads the Form, the new data will be included in the data download.",
      // {icon4} is an icon that displays the number 4. It refers to the fourth step in an image that depicts this workflow.
      "In a future version, you will be able to upload {icon4} a CSV to a Dataset directly.",
      "Datasets are configured during Form design."
    ],
    "getStarted": {
      "full": "For more information on how to get started, {clickHere}.",
      "clickHere": "click here"
    },
    "datasetsWorkflow": "Datasets workflow"
  }
}
</i18n>

<!-- Autogenerated by destructure.js -->
<i18n>
{
  "cs": {
    "title": "V??tejte v n??hledu datov??ch sad!",
    "body": [
      "Datov?? sady p??edstavuj?? nov?? zp??sob vytv????en?? a spr??vy datov??ch p????loh v ODK. Umo????uj?? pou????vat data shrom????d??n?? v Pod??n??ch, kter?? lze st??hnout a pou????t jako sou????st jin??ho formul????e."
    ],
    "datasetsWorkflow": "Pracovn?? postup s datov??mi sadami"
  },
  "fr": {
    "title": "Bienvenue dans l???aper??u des Datasets !",
    "getStarted": {
      "full": "Pour plus d'information sur comment d??marrer, {clickHere}.",
      "clickHere": "cliquez ici"
    }
  }
}
</i18n>
