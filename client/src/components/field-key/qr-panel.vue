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
  <div class="field-key-qr-panel panel panel-default"
    :class="{ legacy: !managed }">
    <div class="panel-heading">
      <h1 class="panel-title">
        {{ managed ? $t('title.managed') : $t('title.legacy') }}
      </h1>
      <span class="icon-mobile"></span>
    </div>
    <div v-if="fieldKey != null" class="panel-body">
      <collect-qr :settings="settings" error-correction-level="L"
        :cell-size="3"/>
      <p>
        <i18n-t v-if="managed" keypath="body[0].managed.full">
          <template #managedCode>
            <strong>{{ $t('body[0].managed.managedCode') }}</strong>
          </template>
        </i18n-t>
        <i18n-t v-else keypath="body[0].legacy.full">
          <template #legacyCode>
            <strong>{{ $t('body[0].legacy.legacyCode') }}</strong>
          </template>
        </i18n-t>
        <sentence-separator/>
        <template v-if="managed">{{ $t('body[1].managed', fieldKey) }}</template>
        <template v-else>{{ $t('body[1].legacy') }}</template>
        <sentence-separator/>
        <i18n-t v-if="managed" keypath="body[2].managed.full">
          <template #switchToLegacy>
            <i18n-t tag="a" keypath="body[2].managed.switchToLegacy"
              class="switch-code" href="#">
              <template #legacyCode>
                <strong>{{ $t('body[2].managed.legacyCode') }}</strong>
              </template>
            </i18n-t>
          </template>
        </i18n-t>
        <i18n-t v-else keypath="body[2].legacy.full">
          <template #switchToManaged>
            <i18n-t tag="a" keypath="body[2].legacy.switchToManaged"
              class="switch-code" href="#">
              <template #managedCode>
                <strong>{{ $t('body[2].legacy.managedCode') }}</strong>
              </template>
            </i18n-t>
          </template>
        </i18n-t>
      </p>
      <p>
        <span>{{ $t('body[3]', fieldKey) }}</span>
        <sentence-separator/>
        <doc-link to="collect-import-export/">{{ $t('moreInfo.learnMore') }}</doc-link>
      </p>
    </div>
  </div>
</template>

<script>
import CollectQr from '../collect-qr.vue';
import DocLink from '../doc-link.vue';
import SentenceSeparator from '../sentence-separator.vue';

import { apiPaths } from '../../util/request';
import { useRequestData } from '../../request-data';

export default {
  name: 'FieldKeyQrPanel',
  components: { CollectQr, DocLink, SentenceSeparator },
  props: {
    fieldKey: Object,
    managed: Boolean
  },
  setup() {
    const { project } = useRequestData();
    return { project };
  },
  computed: {
    settings() {
      const url = apiPaths.serverUrlForFieldKey(
        this.fieldKey.token,
        this.project.id
      );
      const settings = {
        general: { server_url: `${window.location.origin}${url}` },
        project: { name: this.project.name },
        // Collect requires the settings to have an `admin` property.
        admin: {}
      };
      if (this.managed) {
        settings.general.form_update_mode = 'match_exactly';
        settings.general.autosend = 'wifi_and_cellular';
      }
      return settings;
    }
  }
};
</script>

<style lang="scss">
@import '../../assets/scss/variables';

.field-key-qr-panel {
  margin-bottom: 0;
  width: 500px;

  .panel-heading {
    background-color: $color-action-background;
    position: relative;
  }

  // The icon is not animated, because in the popover, switching between a
  // managed and a legacy QR code would reset the animation.
  .icon-mobile {
    color: $color-action-overlay;
    font-size: 98px;
    position: absolute;
    right: 18px;
    top: -32px;
    transform: rotate(15deg);

    &::after {
      background-color: #fff;
      content: '';
      height: 65px;
      left: 5px;
      position: absolute;
      top: 18px;
      width: 32px;
      z-index: -1;
    }
  }

  .collect-qr {
    float: left;
    margin-right: 15px;
  }

  p {
    &:first-child { margin-top: 5px; }
    &:last-child { margin-bottom: 5px; }
  }

  &.legacy {
    .panel-heading { background-color: #777; }
    .icon-mobile { color: #555; }
  }
}
</style>

<i18n lang="json5">
{
  "en": {
    // This is the title at the top of a pop-up. "Client" refers to a data
    // collection client like ODK Collect. "Code" refers to a QR code.
    "title": {
      "managed": "Client Configuration Code",
      "legacy": "Legacy Client Configuration Code"
    },
    "body": [
      {
        "managed": {
          "full": "This is a {managedCode}.",
          "managedCode": "Managed QR Code"
        },
        "legacy": {
          "full": "This is a {legacyCode}.",
          "legacyCode": "Legacy QR Code"
        }
      },
      {
        // "Get Blank Form" is the text of a button in ODK Collect.
        "managed": "Collect will exactly match the Forms available to ???{displayName}??? including automatically applying updates. Users will not need to manually Get Blank Forms. Additionally, finalized Forms will be sent automatically as soon as a connection is found.",
        // "Get Blank Form" and "Send Finalized Form" are the text of buttons in ODK Collect.
        "legacy": "Users will have to manually Get Blank Forms on the device and determine which Forms to update. They will also need to manually Send Finalized Forms."
      },
      {
        "managed": {
          "full": "For the old behavior, {switchToLegacy}.",
          "switchToLegacy": "switch to a {legacyCode}",
          "legacyCode": "Legacy QR Code"
        },
        "legacy": {
          "full": "For a more controlled and foolproof process, {switchToManaged}.",
          "switchToManaged": "switch to a {managedCode}",
          "managedCode": "Managed QR Code"
        }
      },
      "Scan this QR code to configure a device with the account ???{displayName}???."
    ]
  }
}
</i18n>

<!-- Autogenerated by destructure.js -->
<i18n>
{
  "cs": {
    "title": {
      "managed": "Konfigura??n?? k??d klienta",
      "legacy": "Konfigura??n?? k??d star????ho klienta"
    },
    "body": [
      {
        "managed": {
          "full": "Toto je {managedCode}.",
          "managedCode": "Spravovan?? QR k??d"
        },
        "legacy": {
          "full": "Toto je {legacyCode}.",
          "legacyCode": "Star???? QR k??d"
        }
      },
      {
        "managed": "Collect bude p??esn?? odpov??dat dostupn??m formul??????m na ???{displayName}???, v??etn?? automatick??ho pou??it?? aktualizac??. U??ivatel?? nebudou muset ru??n?? z??sk??vat pr??zdn?? formul????e. Dokon??en?? formul????e budou nav??c automaticky odesl??ny, jakmile bude nalezeno spojen??.",
        "legacy": "U??ivatel?? budou muset ru??n?? z??skat pr??zdn?? formul????e v za????zen?? a ur??it, kter?? formul????e se maj?? aktualizovat. Budou tak?? muset ru??n?? odeslat kone??n?? formul????e."
      },
      {
        "managed": {
          "full": "Pro star?? chov??n??, {switchToLegacy}.",
          "switchToLegacy": "p??epn??te na {legacyCode}",
          "legacyCode": "Star???? QR k??d"
        },
        "legacy": {
          "full": "Pro v??ce kontrolovan?? a spolehliv?? proces, {switchToManaged}.",
          "switchToManaged": "p??epn??te na {managedCode}",
          "managedCode": "Spravovan?? QR k??d"
        }
      },
      "Naskenov??n??m tohoto QR k??du nakonfigurujete za????zen?? s ????tem ???{displayName}???."
    ]
  },
  "de": {
    "title": {
      "managed": "Client Configuration Code",
      "legacy": "Legacy Client Configuration Code"
    },
    "body": [
      {
        "managed": {
          "full": "Das ist ein {managedCode}.",
          "managedCode": "Managed QR-Code"
        },
        "legacy": {
          "full": "Das ist ein {legacyCode}.",
          "legacyCode": "Legacy QR-Code"
        }
      },
      {
        "managed": "Collect wird die f??r \"{displayName}\" verf??gbaren Formulare genau abgleichen und automatisch Updates anwenden. Benutzer m??ssen Leerformulare nicht manuell herunterladen. Au??erdem werden abgeschlossene Formulare automatisch ??bermittelt sobald eine Verbindung vorhanden ist.",
        "legacy": "Benutzer m??ssen manuell auf dem Ger??t Leerformulare herunterladen und Formulare zum Updaten ausw??hlen. Sie m??ssen au??erdem manuell vollst??ndige Formulare hochladen."
      },
      {
        "managed": {
          "full": "F??r das alte Verhalten {switchToLegacy}.",
          "switchToLegacy": "auf {legacyCode} umschalten",
          "legacyCode": "Legacy QR-Code"
        },
        "legacy": {
          "full": "F??r einen kontrollierteren und idiotensicheren Prozess {switchToManaged}.",
          "switchToManaged": "auf {managedCode} umschalten",
          "managedCode": "Managed QR-Code"
        }
      },
      "Scannen Sie diesen QR-Code, um ein Ger??t mit dem Konto \"{displayName}\" zu konfigurieren."
    ]
  },
  "es": {
    "title": {
      "managed": "C??digo de configuraci??n del cliente",
      "legacy": "C??digo de configuraci??n de cliente heredado"
    },
    "body": [
      {
        "managed": {
          "full": "Este es un {managedCode}",
          "managedCode": "C??digo QR gestionado"
        },
        "legacy": {
          "full": "Este es un {legacyCode}",
          "legacyCode": "C??digo QR heredado"
        }
      },
      {
        "managed": "Collect coincidir?? exactamente con los formularios disponibles para {displayName}, incluida la aplicaci??n autom??tica de actualizaciones. Los usuarios no necesitar??n obtener formularios en blanco manualmente. Adem??s, los formularios finalizados se enviar??n autom??ticamente tan pronto como se encuentre una conexi??n.",
        "legacy": "Los usuarios deber??n obtener manualmente formularios en blanco en el dispositivo y determinar qu?? formularios actualizar. Tambi??n necesitar??n enviar formularios finalizados manualmente"
      },
      {
        "managed": {
          "full": "Por el viejo comportamiento, {switchToLegacy}",
          "switchToLegacy": "cambiar a un {legacyCode}",
          "legacyCode": "C??digo QR heredado"
        },
        "legacy": {
          "full": "Para un proceso m??s controlado e infalible, {switchToManaged}",
          "switchToManaged": "cambiar a un {managedCode}",
          "managedCode": "C??digo QR gestionado"
        }
      },
      "Escanee este c??digo QR para configurar un dispositivo con la cuenta \"{displayName}\""
    ]
  },
  "fr": {
    "title": {
      "managed": "Code de configuration du client.",
      "legacy": "Ancien code de configuration de client"
    },
    "body": [
      {
        "managed": {
          "full": "Ceci est un {managedCode}.",
          "managedCode": "QR Code administr??"
        },
        "legacy": {
          "full": "Ceci est un {legacyCode}.",
          "legacyCode": "Ancien QR Code"
        }
      },
      {
        "managed": "Collect refl??tera exactement les formulaires disponibles sur ???{displayName}??? incluant leurs mises ?? jour automatiques. Les utilisateurs n'auront plus ?? t??l??charger les formulaires vierges. En outre, les formulaires finalis??s seront envoy??s automatiquement, d??s qu'une connexion sera trouv??e.",
        "legacy": "Les utilisateurs devront manuellement t??l??charger les formulaires vierges sur le t??l??phone et choisir quels formulaires mettre ?? jour. Ils devront aussi envoyer les formulaires finalis??s manuellement."
      },
      {
        "managed": {
          "full": "Pour l'ancien comportement, {switchToLegacy}.",
          "switchToLegacy": "Changez pour un {legacyCode}",
          "legacyCode": "ancien QR Code"
        },
        "legacy": {
          "full": "Pour un processus plus contr??l?? et infaillible, {switchToManaged}.",
          "switchToManaged": "Changer pour un {managedCode}",
          "managedCode": "QR Code administr??"
        }
      },
      "Scannez ce QR Code pour configurer un t??l??phone avec le compte ???{displayName}???."
    ]
  },
  "id": {
    "title": {
      "managed": "Kode Konfigurrasi Klien",
      "legacy": "Kode Konfigurasi Klien Versi Lama"
    },
    "body": [
      {
        "managed": {
          "full": "Ini adalah {managedCode}.",
          "managedCode": "Kode QR yang dikelola"
        },
        "legacy": {
          "full": "Ini adalah {legacyCode}.",
          "legacyCode": "kode QR Versi Lama"
        }
      },
      {
        "legacy": "Pengguna harus mendapatkan formulir kosong secara manual di perangkat dan menentukan formulir mana yang akan diperbarui. Pengguna juga harus mengirim formulir yang telah selesai secara manual."
      },
      {
        "managed": {
          "full": "Untuk cara lama, {switchToLegacy}.",
          "switchToLegacy": "beralih ke {legacyCode}",
          "legacyCode": "Kode QR versi Lama"
        },
        "legacy": {
          "full": "Untuk proses yang lebih terkontrol dan terpercaya, {switchToManaged}.",
          "switchToManaged": "beralih ke {managedCode}",
          "managedCode": "Kode QR yang Dikelola"
        }
      },
      "Pindai kode QR ini untuk mengonfigurasi perangkat dengan akun tersebut ???{displayName}???."
    ]
  },
  "it": {
    "title": {
      "managed": "Codice configurazione del Client",
      "legacy": "Codice di configurazione client legacy"
    },
    "body": [
      {
        "managed": {
          "full": "Questo ?? un {managedCode}.",
          "managedCode": "Codice QR gestito"
        },
        "legacy": {
          "full": "Questo ?? un {legacyCode}.",
          "legacyCode": "Codice QR legacy"
        }
      },
      {
        "managed": "Collect corrisponder?? esattamente ai formulari disponibili per \"{displayName}\", inclusa l'applicazione automatica degli aggiornamenti. Gli utenti non dovranno recuperare manualmente i formulari vuoti. Inoltre, i formulari finalizzati verranno inviati automaticamente non appena viene trovata una connessione.",
        "legacy": "Gli utenti dovranno recuperare manualmente i formulari vuoti sul dispositivo e stabilire quali formulari aggiornare. Dovranno anche inviare manualmente i formulari finalizzati."
      },
      {
        "managed": {
          "full": "Per il vecchio modo di operare, {switchToLegacy}.",
          "switchToLegacy": "passare a {legacyCode}",
          "legacyCode": "Codice QR legacy"
        },
        "legacy": {
          "full": "Per un processo pi?? controllato e prova d'errore,{switchToManaged}.",
          "switchToManaged": "passare a {managedCode}",
          "managedCode": "Codice QR gestito"
        }
      },
      "Scansiona questo codice QR per configurare un dispositivo con l'account \"{displayName}\"."
    ]
  },
  "ja": {
    "title": {
      "managed": "????????????????????????????????????",
      "legacy": "??????????????????????????????????????????"
    },
    "body": [
      {
        "managed": {
          "full": "?????????{managedCode}?????????",
          "managedCode": "????????????QR?????????"
        },
        "legacy": {
          "full": "?????????{legacyCode}?????????",
          "legacyCode": "????????????QR?????????"
        }
      },
      {
        "managed": "ODK Collect?????????{displayName}?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????",
        "legacy": "???????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????"
      },
      {
        "managed": {
          "full": "???????????????????????????????????????{switchToLegacy}",
          "switchToLegacy": "{legacyCode}?????????????????????",
          "legacyCode": "????????????QR?????????"
        },
        "legacy": {
          "full": "???????????????????????????????????????????????????{switchToManaged}",
          "switchToManaged": "{managedCode}?????????????????????",
          "managedCode": "????????????QR?????????"
        }
      },
      "??????QR???????????????????????????????????????????????????\"{displayName}\"???????????????????????????"
    ]
  },
  "sw": {
    "title": {
      "managed": "Msimbo wa Usanidi wa Mteja",
      "legacy": "Msimbo wa Usanidi wa Mteja wa Urithi"
    },
    "body": [
      {
        "managed": {
          "full": "Hii ni {managedCode}",
          "managedCode": "Msimbo wa QR unaosimamiwa"
        },
        "legacy": {
          "full": "Hii ni {legacyCode}.",
          "legacyCode": "Msimbo wa QR wa urithi"
        }
      },
      {
        "managed": "Mkusanyiko utalingana kabisa na Fomu zinazopatikana kwa \"{displayName}\" ikijumuisha kutumia masasisho kiotomatiki. Watumiaji hawatahitaji Kujipatia Fomu tupu. Zaidi ya hayo, Fomu zilizokamilishwa zitatumwa kiotomatiki mara tu muunganisho utakapopatikana.",
        "legacy": "Watumiaji watalazimika Kupata mwenyewe Fomu tupu kwenye kifaa na kuamua ni Fomu zipi za kusasisha. Pia watahitaji Kutuma mwenyewe Fomu Zilizokamilishwa"
      },
      {
        "managed": {
          "full": "Kwa tabia ya zamani, {switchToLegacy}",
          "switchToLegacy": "badilisha hadi {legacyCode}",
          "legacyCode": "Msimbo wa QR wa urithi"
        },
        "legacy": {
          "full": "Kwa mchakato unaodhibitiwa zaidi na usiofaa, {switchToManaged}",
          "switchToManaged": "badilisha hadi {managedCode}",
          "managedCode": "Msimbo wa QR unaosimamiwa"
        }
      },
      "Changanua msimbo huu wa QR ili kusanidi kifaa kilicho na akaunti \"{displayName}\"."
    ]
  }
}
</i18n>
