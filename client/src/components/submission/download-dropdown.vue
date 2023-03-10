<!--
Copyright 2020 ODK Central Developers
See the NOTICE file at the top-level directory of this distribution and at
https://github.com/getodk/central-frontend/blob/master/NOTICE.

This file is part of ODK Central. It is subject to the license terms in
the LICENSE file found in the top-level directory of this distribution and at
https://www.apache.org/licenses/LICENSE-2.0. No part of ODK Central,
including this file, may be copied, modified, propagated, or distributed
except according to the terms contained in the LICENSE file.
-->
<template>
  <button id="submission-download-button" type="button" class="btn btn-primary"
    @click="$emit('download')">
    <span class="icon-arrow-circle-down"></span>
    <span>{{ text }}&hellip;</span>
  </button>
</template>

<script>
import { useRequestData } from '../../request-data';

export default {
  // Ideally, this component would be named SubmissionDownloadButton, but that
  // would cause messages to be moved in Transifex.
  name: 'SubmissionDownloadDropdown',
  props: {
    formVersion: Object,
    filtered: Boolean
  },
  emits: ['download'],
  setup() {
    const { odata } = useRequestData();
    return { odata };
  },
  computed: {
    text() {
      if (!this.filtered) {
        if (!this.formVersion.dataExists) return '';
        return this.$tcn(
          'action.download.unfiltered',
          this.formVersion.submissions
        );
      }

      return !this.odata.dataExists
        ? this.$t('action.download.filtered.withoutCount')
        : this.$tcn('action.download.filtered.withCount', this.odata.count);
    }
  }
};
</script>

<i18n lang="json5">
{
  "en": {
    "action": {
      "download": {
        "unfiltered": "Download {count} Submission | Download {count} Submissions",
        "filtered": {
          // This is the text of a button. This text is shown when the number of
          // matching Submissions is unknown.
          "withoutCount": "Download matching Submissions",
          "withCount": "Download {count} matching Submission | Download {count} matching Submissions"
        }
      }
    }
  }
}
</i18n>

<!-- Autogenerated by destructure.js -->
<i18n>
{
  "cs": {
    "action": {
      "download": {
        "filtered": {
          "withoutCount": "St??hnout odpov??daj??c?? p????sp??vky"
        }
      }
    }
  },
  "de": {
    "action": {
      "download": {
        "unfiltered": "{count} ??bermittlung herunterladen | {count} ??bermittlungen herunterladen",
        "filtered": {
          "withoutCount": "Passende ??bermittlungen herunterladen",
          "withCount": "{count} passende ??bermittlung herunterladen | {count} passende ??bermittlungen herunterladen"
        }
      }
    }
  },
  "es": {
    "action": {
      "download": {
        "filtered": {
          "withoutCount": "Descargar env??os coincidentes."
        }
      }
    }
  },
  "fr": {
    "action": {
      "download": {
        "unfiltered": "T??l??charger {count} soumission | T??l??charger {count} soumissions | T??l??charger {count} soumission(s)",
        "filtered": {
          "withoutCount": "T??l??charger les soumissions correspondantes",
          "withCount": "T??l??charger {count} soumission correspondant | T??l??charger {count} soumissions correspondantes | T??l??charger {count} souission(s) correspondant(s)"
        }
      }
    }
  },
  "it": {
    "action": {
      "download": {
        "unfiltered": "Scarica {count} Invio | Scarica {count} Invii | Scarica {count} Invii",
        "filtered": {
          "withoutCount": "Scarica invii corrispondenti.",
          "withCount": "Scarica {count} invio corrispondente. | Scarica {count} invii corrispondenti. | Scarica {count} invii corrispondenti."
        }
      }
    }
  }
}
</i18n>
