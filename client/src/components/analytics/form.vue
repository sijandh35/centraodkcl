<!--
Copyright 2021 ODK Central Developers
See the NOTICE file at the top-level directory of this distribution and at
https://github.com/getodk/central-frontend/blob/master/NOTICE.

This file is part of ODK Central. It is subject to the license terms in
the LICENSE file found in the top-level directory of this distribution and at
https://www.apache.org/licenses/LICENSE-2.0. No part of ODK Central,
including this file, may be copied, modified, propagated, or distributed
except according to the terms contained in the LICENSE file.
-->
<template>
  <form id="analytics-form" @submit.prevent="submit">
    <div class="radio">
      <label>
        <input v-model="enabled" type="radio" :value="null"
          aria-describedby="analytics-form-enabled-null-help">
        <strong>{{ $t('enabled.null[0]') }}</strong>
      </label>
      <p id="analytics-form-enabled-null-help" class="help-block">
        {{ $t('enabled.null[1]') }}
      </p>
    </div>
    <div class="radio">
      <label>
        <input v-model="enabled" type="radio" :value="false"
          aria-describedby="analytics-form-enabled-false-help">
        <strong>{{ $t('enabled.false[0]') }}</strong>
      </label>
      <p id="analytics-form-enabled-false-help" class="help-block">
        {{ $t('enabled.false[1]') }}
      </p>
    </div>
    <div class="radio">
      <label>
        <input v-model="enabled" type="radio" :value="true">
        <i18n-t keypath="enabled.true[0].full">
          <template #weWillShare>
            <strong>{{ $t('enabled.true[0].weWillShare') }}</strong>
          </template>
          <template #termsOfService>
            <a href="https://getodk.org/legal/tos.html" target="_blank">{{ $t('enabled.true[0].termsOfService') }}</a>
          </template>
          <template #privacyPolicy>
            <a href="https://getodk.org/legal/privacy.html" target="_blank">{{ $t('enabled.true[0].privacyPolicy') }}</a>
          </template>
        </i18n-t>
      </label>
      <p id="analytics-form-enabled-true-help" class="help-block">
        <a href="#" @click.prevent="$emit('preview')">
          <span class="icon-question-circle-o"></span>{{ $t('enabled.true[1]') }}
        </a>
      </p>
    </div>
    <fieldset :disabled="enabled !== true">
      <div class="checkbox">
        <label>
          <input v-model="contact" type="checkbox"
            aria-describedby="analytics-form-contact-help">
          <strong>{{ $t('contact[0]') }}</strong>
        </label>
        <p id="analytics-form-contact-help" class="help-block">
          {{ $t('contact[1]') }}
        </p>
      </div>
      <fieldset :disabled="!contact">
        <form-group v-model.trim="email" type="email"
          :placeholder="$t('field.workEmail')" required autocomplete="off"/>
        <form-group v-model.trim="organization"
          :placeholder="$t('field.organization')" autocomplete="organization"/>
      </fieldset>
    </fieldset>
    <button type="submit" class="btn btn-primary" :disabled="awaitingResponse">
      {{ $t('action.saveSettings') }} <spinner :state="awaitingResponse"/>
    </button>
  </form>
</template>

<script>
import FormGroup from '../form-group.vue';
import Spinner from '../spinner.vue';

import { noop } from '../../util/util';
import { useRequestData } from '../../request-data';

export default {
  name: 'AnalyticsForm',
  components: { FormGroup, Spinner },
  inject: ['alert'],
  emits: ['preview'],
  setup() {
    const { currentUser, analyticsConfig } = useRequestData();
    const { awaitingResponse } = analyticsConfig.toRefs();
    return { currentUser, analyticsConfig, awaitingResponse };
  },
  data() {
    const configValue = this.analyticsConfig
      .map(({ value }) => value)
      .orElseGet(() => ({ enabled: null }));
    return {
      enabled: configValue.enabled,
      contact: configValue.email != null || configValue.organization != null,
      email: configValue.email != null
        ? configValue.email
        : (configValue.organization == null ? this.currentUser.email : ''),
      organization: configValue.organization != null
        ? configValue.organization
        : ''
    };
  },
  methods: {
    setConfig() {
      const postData = { enabled: this.enabled };
      if (this.enabled && this.contact) {
        if (this.email !== '') postData.email = this.email;
        if (this.organization !== '')
          postData.organization = this.organization;
      }
      return this.analyticsConfig.request({
        method: 'POST',
        url: '/v1/config/analytics',
        data: postData
      });
    },
    unsetConfig() {
      return this.analyticsConfig.request({
        method: 'DELETE',
        url: '/v1/config/analytics',
        patch: () => { this.analyticsConfig.setToNone(); }
      });
    },
    submit() {
      (this.enabled != null ? this.setConfig() : this.unsetConfig())
        .then(() => {
          this.alert.success(this.$t('alert.success'));
        })
        .catch(noop);
    }
  }
};
</script>

<style lang="scss">
@import '../../assets/scss/variables';

#analytics-form {
  margin-bottom: 35px;

  .radio { margin-bottom: 21px; }
  .help-block { color: $color-text; }
  fieldset { padding-left: 20px; }
  > fieldset { margin-top: -6px; }
  fieldset fieldset { margin-bottom: 5px; }
  .form-control { width: 375px; }
}

#analytics-form-enabled-true-help { margin-top: 12px; }
</style>

<i18n lang="json5">
{
  "en": {
    "enabled": {
      "null": [
        "Remind us later.",
        "Administrators will continue to see the message at the top of the screen."
      ],
      "true": [
        {
          "full": "{weWillShare} and we accept the {termsOfService} and {privacyPolicy}.",
          "weWillShare": "We are willing to share anonymous usage data monthly with the Central team,",
          "termsOfService": "Terms of Service",
          "privacyPolicy": "Privacy Policy"
        },
        "What metrics are sent?"
      ],
      "false": [
        "We are not interested in sharing any information.",
        "You won???t see a reminder about this again."
      ]
    },
    "contact": [
      "I am willing to include my contact information with the report.",
      "We may contact you to learn more about your usage of Central."
    ],
    "field": {
      "workEmail": "Work email address",
      "organization": "Organization name"
    },
    "alert": {
      "success": "Settings saved!"
    }
  }
}
</i18n>

<!-- Autogenerated by destructure.js -->
<i18n>
{
  "cs": {
    "enabled": {
      "null": [
        "P??ipome??te n??m to pozd??ji.",
        "Spr??vci budou i nad??le zobrazovat zpr??vu v horn?? ????sti obrazovky."
      ],
      "true": [
        {
          "full": "{weWillShare} a p??ij??m??me {termsOfService} a {privacyPolicy}.",
          "weWillShare": "Jsme ochotni m??s????n?? sd??let anonymn?? ??daje o pou????v??n?? s centr??ln??m t??mem,",
          "termsOfService": "Podm??nky slu??by",
          "privacyPolicy": "Z??sady ochrany osobn??ch ??daj??"
        },
        "Jak?? metriky se odes??laj???"
      ],
      "false": [
        "Nem??me z??jem sd??let ????dn?? informace.",
        "Tuto p??ipom??nku u?? neuvid??te."
      ]
    },
    "contact": [
      "Jsem ochoten p??ipojit ke zpr??v?? sv?? kontaktn?? ??daje.",
      "M????eme v??s kontaktovat, abychom se dozv??d??li v??ce o va??em pou????v??n?? slu??by Central."
    ],
    "field": {
      "workEmail": "Pracovn?? e-mailov?? adresa",
      "organization": "N??zev organizace"
    },
    "alert": {
      "success": "Nastaven?? ulo??eno!"
    }
  },
  "de": {
    "enabled": {
      "null": [
        "Erinnere uns sp??ter",
        "Administratoren werden die Nachrichten weiterhin im oberen Bereich des Bildschirms sehen."
      ],
      "true": [
        {
          "full": "{weWillShare} und wir akzeptieren die {termsOfService} und {privacyPolicy}.",
          "weWillShare": "Wir sind einverstanden, anonyme Nutzungsdaten monatlich mit dem Central-Team zu teilen.",
          "termsOfService": "Nutzungsbedingungen",
          "privacyPolicy": "Datenschutzerkl??rung"
        },
        "Welche Messwerte werden ??bermittelt?"
      ],
      "false": [
        "Wir sind nicht daran interessiert Informationen zu teilen.",
        "Sie werden keine Erinnerung mehr kriegen."
      ]
    },
    "contact": [
      "Ich bin einverstanden, dass meine Kontaktdaten im Report ersichtlich sind.",
      "Wir werden Sie unter Umst??nden kontaktieren, um mehr ??ber Ihre Nutzung von Central zu erfahren."
    ],
    "field": {
      "workEmail": "Email Adresse (gesch??ftlich)",
      "organization": "Name der Organisation"
    },
    "alert": {
      "success": "Einstellungen gespeichert!"
    }
  },
  "es": {
    "enabled": {
      "null": [
        "Recu??rdanos m??s tarde.",
        "Los administradores seguir??n viendo el mensaje en la parte superior de la pantalla."
      ],
      "true": [
        {
          "full": "{weWillShare} y aceptamos los {termsOfService} y {privacyPolicy}",
          "weWillShare": "Estamos dispuestos a compartir mensualmente datos de uso an??nimos con el equipo central,",
          "termsOfService": "T??rminos del servicio",
          "privacyPolicy": "Pol??tica de privacidad"
        },
        "??Qu?? m??tricas se env??an?"
      ],
      "false": [
        "No estamos interesados en compartir ninguna informaci??n.",
        "No volver?? a ver un recordatorio sobre esto."
      ]
    },
    "contact": [
      "Estoy dispuesto a incluir mi informaci??n de contacto con el informe.",
      "Es posible que nos comuniquemos con usted para obtener m??s informaci??n sobre su uso de Central."
    ],
    "field": {
      "workEmail": "Direcci??n de correo electr??nico del trabajo",
      "organization": "Nombre de la Organizaci??n"
    },
    "alert": {
      "success": "??Configuraci??n guardada!"
    }
  },
  "fr": {
    "enabled": {
      "null": [
        "Nous le rappeler plus tard.",
        "Les administrateurs continueront ?? voir le message en haut de l'??cran."
      ],
      "true": [
        {
          "full": "{weWillShare} et nous acceptons les {termsOfService} et la {privacyPolicy}.",
          "weWillShare": "Nous souhaitons partager mensuellement des donn??es d'usage anonymis??es avec l?????quipe de Central .",
          "termsOfService": "Conditions de service",
          "privacyPolicy": "Politique de confidentialit??"
        },
        "Quelles sont les m??triques envoy??es ?"
      ],
      "false": [
        "Nous ne sommes pas int??ress??s par le partage de ces informations.",
        "Vous ne verrez plus de rappel ?? ce sujet."
      ]
    },
    "contact": [
      "Je souhaite inclure mes informations de contact dans le rapport.",
      "Nous pouvons vous contacter pour en savoir plus sur votre utilisation de Central."
    ],
    "field": {
      "workEmail": "Adresse de courriel professionnelle",
      "organization": "Nom de l'organisation"
    },
    "alert": {
      "success": "R??glages enregistr??s."
    }
  },
  "id": {
    "alert": {
      "success": "Pengaturan disimpan!"
    }
  },
  "it": {
    "enabled": {
      "null": [
        "Ricordacelo dopo",
        "Gli amministratori continueranno a vedere il messaggio nella parte superiore dello schermo."
      ],
      "true": [
        {
          "full": "{weWillShare} e accettiamo i {termsOfService} e {privacyPolicy}.",
          "weWillShare": "Siamo disposti a condividere mensilmente dati di utilizzo anonimi con il team Central,",
          "termsOfService": "Termini di servizio",
          "privacyPolicy": "Politica sulla Privacy."
        },
        "Quali metriche vengono inviate?"
      ],
      "false": [
        "Non siamo interessati a condividere alcuna informazione.",
        "Non vedrai pi?? un promemoria al riguardo."
      ]
    },
    "contact": [
      "Sono disposto a includere le mie informazioni di contatto con il rapporto.",
      "Potremmo contattarti per capire meglio sul tuo utilizzo di Central"
    ],
    "field": {
      "workEmail": "Indirizzo email del lavoro",
      "organization": "Nome dell'Organizzazione"
    },
    "alert": {
      "success": "Impostazioni salvate!"
    }
  },
  "ja": {
    "enabled": {
      "null": [
        "?????????????????????",
        "??????????????????????????????????????????????????????????????????????????????????????????"
      ],
      "true": [
        {
          "full": "{weWillShare}?????????{termsOfService}???{privacyPolicy}?????????????????????",
          "weWillShare": "??????????????????????????????????????????????????????Central??????????????????????????????",
          "termsOfService": "????????????",
          "privacyPolicy": "??????????????????????????????"
        },
        "????????????????????????????????????????????????"
      ],
      "false": [
        "????????????????????????????????????????????????????????????????????????",
        "?????????????????????????????????????????????????????????????????????????????????"
      ]
    },
    "contact": [
      "?????????????????????????????????????????????????????????",
      "???????????????????????????ODK Central?????????????????????????????????????????????????????????????????????"
    ],
    "field": {
      "workEmail": "??????????????????????????????",
      "organization": "?????????"
    },
    "alert": {
      "success": "?????????????????????????????????"
    }
  },
  "sw": {
    "enabled": {
      "null": [
        "Tukumbushe baadaye",
        "Wasimamizi wataendelea kuona ujumbe juu ya skrini"
      ],
      "true": [
        {
          "full": "{weWillShare} na tunakubali {termsOfService} na {privacyPolicy}.",
          "weWillShare": "Tuko tayari kushiriki data ya matumizi isiyojulikana kila mwezi na timu ya Kati",
          "termsOfService": "Masharti ya Huduma",
          "privacyPolicy": "Sera ya Faragha"
        },
        "Ni vipimo gani vinatumwa?"
      ],
      "false": [
        "Hatutashiriki habari yoyote.",
        "hutaona kikumbusho kuhusu hili tena"
      ]
    },
    "contact": [
      "Niko tayari kujumuisha maelezo yangu ya mawasiliano na ripoti",
      "Tunaweza kuwasiliana nawe ili kupata maelezo zaidi kuhusu matumizi yako ya Central"
    ],
    "field": {
      "workEmail": "Anwani ya barua pepe ya kazini",
      "organization": "Jina la shirika"
    },
    "alert": {
      "success": "Mipangilio imehifadhiwa!"
    }
  }
}
</i18n>
