<!--
Copyright 2019 ODK Central Developers
See the NOTICE file at the top-level directory of this distribution and at
https://github.com/getodk/central-frontend/blob/master/NOTICE.

This file is part of ODK Central. It is subject to the license terms in
the LICENSE file found in the top-level directory of this distribution and at
https://www.apache.org/licenses/LICENSE-2.0. No part of ODK Central,
including this file, may be copied, modified, propagated, or distributed
except according to the terms contained in the LICENSE file.
-->
<template>
  <div id="backup-status">
    <span id="backup-status-icon" :class="iconClass"></span>

    <template v-if="status === 'notConfigured'">
      <p>{{ $t('notConfigured[0]') }}</p>
      <p>
        <span>{{ $t('notConfigured[1]') }}</span>
        <sentence-separator/>
        <i18n-t keypath="moreInfo.clickHere.full">
          <template #clickHere>
            <a href="https://forum.getodk.org/t/backups-to-google-drive-from-central-will-stop-working-after-jan-31st/38895" target="_blank">{{ $t('moreInfo.clickHere.clickHere') }}</a>
          </template>
        </i18n-t>
      </p>
    </template>
    <template v-else-if="status === 'neverRun'">
      <p>{{ $t('neverRun[0]') }}</p>
      <p>{{ $t('neverRun[1]') }}</p>
      <p>
        <span>{{ $t('neverRun[2]') }}</span>
        <sentence-separator/>
        <i18n-t keypath="getHelp.full">
          <template #forum>
            <a href="https://forum.getodk.org/" target="_blank">{{ $t('getHelp.forum') }}</a>
          </template>
        </i18n-t>
      </p>
    </template>
    <template v-else-if="status === 'somethingWentWrong'">
      <p>{{ $t('somethingWentWrong[0]') }}</p>
      <i18n-t tag="p" keypath="somethingWentWrong[1].full">
        <template #moreThanThreeDaysAgo>
          <strong>{{ $t('somethingWentWrong[1].moreThanThreeDaysAgo') }}</strong>
        </template>
      </i18n-t>
      <p>
        <span>{{ $t('somethingWentWrong[2]') }}</span>
        <sentence-separator/>
        <i18n-t keypath="getHelp.full">
          <template #forum>
            <a href="https://forum.getodk.org/" target="_blank">{{ $t('getHelp.forum') }}</a>
          </template>
        </i18n-t>
      </p>
    </template>
    <template v-else>
      <p>{{ $t('success[0]') }}</p>
      <i18n-t tag="p" keypath="success[1]">
        <template #dateTime>
          <strong id="backup-status-most-recently-logged-at">
            <date-time :iso="latestAudit.loggedAt"/>
          </strong>
        </template>
      </i18n-t>
    </template>

    <div v-if="status !== 'notConfigured'">
      <a class="btn btn-primary" :class="{ disabled: downloading }"
        href="/v1/backup" @click="download">
        <span class="icon-arrow-circle-down"></span>{{ $t('action.download') }}
        <spinner :state="downloading"/>
      </a>
      <button type="button" class="btn btn-danger" @click="$emit('terminate')">
        <span class="icon-times-circle"></span>{{ $t('action.terminate') }}&hellip;
      </button>
    </div>
  </div>
</template>

<script>
import { DateTime } from 'luxon';

import DateTimeComponent from '../date-time.vue';
import SentenceSeparator from '../sentence-separator.vue';
import Spinner from '../spinner.vue';

import { ago } from '../../util/date-time';
import { useRequestData } from '../../request-data';

export default {
  name: 'BackupStatus',
  components: { DateTime: DateTimeComponent, SentenceSeparator, Spinner },
  inject: ['alert'],
  emits: ['terminate'],
  setup() {
    // The component assumes that this data will exist when the component is
    // is created.
    const { backupsConfig, audits } = useRequestData();
    return { backupsConfig, audits };
  },
  data() {
    return {
      downloading: false
    };
  },
  computed: {
    // Returns the latest backup attempt for the current backups config.
    latestAudit() {
      if (!(this.audits.dataExists && this.backupsConfig.dataExists))
        return null;
      if (this.backupsConfig.isEmpty()) return null;
      for (const audit of this.audits) {
        if (audit.loggedAt < this.backupsConfig.get().setAt) {
          // Any backup attempts that follow are for a previous config:
          // this.audits is sorted descending by loggedAt.
          return null;
        }

        const { details } = audit;
        /* This will evaluate to `false` only if an attempt for a previous
        config was logged after the current config was created, which seems
        unlikely. A failed attempt might not have a configSetAt property, which
        means that if a failed attempt was logged after the current config was
        created, we might not be able to determine whether the attempt
        corresponds to the current config or (again unlikely) to a previous one.
        We assume that an attempt without a configSetAt property corresponds to
        the current config. */
        if (details.configSetAt === this.backupsConfig.get().setAt ||
          details.configSetAt == null)
          return audit;
      }
      return null;
    },
    status() {
      if (this.backupsConfig.isEmpty()) return 'notConfigured';
      // The earliest DateTime that is still considered recent for the purposes
      // of this component
      const recentThreshold = ago({ days: 3 });
      // No recent backup attempt
      if (this.latestAudit == null ||
        DateTime.fromISO(this.latestAudit.loggedAt) < recentThreshold) {
        const setAt = DateTime.fromISO(this.backupsConfig.get().setAt);
        return setAt < recentThreshold ? 'somethingWentWrong' : 'neverRun';
      }
      return this.latestAudit.details.success ? 'success' : 'somethingWentWrong';
    },
    iconClass() {
      switch (this.status) {
        case 'notConfigured':
          return 'icon-question-circle';
        case 'neverRun':
        case 'success':
          return 'icon-check-circle';
        default: // 'somethingWentWrong'
          return 'icon-times-circle';
      }
    }
  },
  methods: {
    download(event) {
      if (!this.downloading) {
        this.alert.success(this.$t('alert.download'));
        this.downloading = true;
      } else {
        event.preventDefault();
      }
    }
  }
};
</script>

<style lang="scss">
@import '../../assets/scss/variables';

#backup-status {
  margin-bottom: 35px;
  position: relative;

  .btn + .btn { margin-left: 5px; }
}

#backup-status-icon {
  font-size: 32px;
  position: absolute;
  top: 4px;

  &.icon-question-circle { color: #999; }
  &.icon-check-circle { color: $color-success; }
  &.icon-times-circle { color: $color-danger; }

  ~ * { margin-left: 41px; }
  + p { font-size: 28px; }
}
</style>

<i18n lang="json5">
{
  "en": {
    "getHelp": {
      "full": "If you are having trouble, please try the {forum}.",
      "forum": "community forum"
    },
    "notConfigured": [
      "Backups are not configured.",
      "In earlier versions of ODK Central, it was possible to set up automatic backups to Google Drive. That feature has been deprecated, but there are other ways to back up the data on the server. We strongly recommend that all users back up their data."
    ],
    "neverRun": [
      "The configured backup has not yet run.",
      "If you have configured backups within the last couple of days, this is normal. Otherwise, something has gone wrong.",
      "In that case, the most likely fix is to restart the service."
    ],
    "somethingWentWrong": [
      "Something is wrong!",
      {
        "full": "The latest backup that completed successfully was {moreThanThreeDaysAgo}.",
        "moreThanThreeDaysAgo": "more than three days ago"
      },
      "The most likely fix is to restart the service."
    ],
    "success": [
      // This text is displayed if the latest backup attempt was successful. It indicates that the backup process is working.
      "Backup is working.",
      // {dateTime} shows the date and time at which the last backup completed,
      // for example: "2020/01/01 01:23". It may show a formatted date like
      // "2020/01/01", or it may use a word like "today", "yesterday", or
      // "Sunday". {dateTime} is formatted in bold.
      "The last backup completed successfully {dateTime}."
    ],
    "action": {
      "download": "Download backup now",
      "terminate": "Terminate"
    },
    "alert": {
      "download": "The backup is running now, and will be encrypted and downloaded to your computer. This may take a while. Once the download begins, you can leave this page."
    }
  }
}
</i18n>

<!-- Autogenerated by destructure.js -->
<i18n>
{
  "cs": {
    "getHelp": {
      "full": "Pokud m??te pot????e, zkuste pros??m {forum}.",
      "forum": "komunitn?? f??rum"
    },
    "notConfigured": [
      "Z??lohy nejsou nakonfigurov??ny.",
      "V d????v??j????ch verz??ch ODK Central bylo mo??n?? nastavit automatick?? z??lohov??n?? na Disk Google. Tato funkce byla zru??ena, ale existuj?? jin?? zp??soby z??lohov??n?? dat na serveru. V??em u??ivatel??m d??razn?? doporu??ujeme, aby si sv?? data z??lohovali."
    ],
    "neverRun": [
      "Nakonfigurovan?? z??lohov??n?? dosud nebylo spu??t??no.",
      "Pokud jste nakonfigurovali z??lohov??n?? v posledn??ch n??kolika dnech, je to norm??ln??. Jinak se n??co pokazilo.",
      "V takov??m p????pad?? je nejpravd??podobn??j???? opravou restartov??n?? slu??by."
    ],
    "somethingWentWrong": [
      "N??co je ??patn??!",
      {
        "full": "Posledn?? ??sp????n?? z??loha byla {moreThanThreeDaysAgo}.",
        "moreThanThreeDaysAgo": "p??ed v??ce ne?? t??emi dny"
      },
      "Nejpravd??podobn??j???? opravou je restartov??n?? slu??by."
    ],
    "success": [
      "Z??lohov??n?? funguje.",
      "Posledn?? z??loha byla ??sp????n?? dokon??ena {dateTime}."
    ],
    "action": {
      "download": "St??hnout z??lohu hned",
      "terminate": "Ukon??it"
    },
    "alert": {
      "download": "Z??loha nyn?? b?????? a bude ??ifrov??na a sta??ena do va??eho po????ta??e. M????e to chv??li trvat. Jakmile za??ne stahov??n??, m????ete tuto str??nku opustit."
    }
  },
  "de": {
    "getHelp": {
      "full": "Wenn es nicht funktioniert, fragen Sie im {forum} nach.",
      "forum": "Community-Forum"
    },
    "notConfigured": [
      "Sicherungskopien sind nicht konfiguriert."
    ],
    "neverRun": [
      "Die konfigurierte Sicherung ist wurde noch nicht ausgef??hrt.",
      "Wenn Sie die Sicherungen in den letzten Tagen konfiguriert haben, ist dies normal. Wenn nicht, ist irgendetwas schief gegangen."
    ],
    "somethingWentWrong": [
      "Irgendetwas hat da nicht geklappt.",
      {
        "full": "Die letzte vollst??ndige Sicherung wurde {moreThanThreeDaysAgo} ausgef??hrt.",
        "moreThanThreeDaysAgo": "vor mehr als drei Tagen."
      }
    ],
    "success": [
      "Die Sicherung wird gerade erstellt.",
      "Die letzte Sicherungskopie wurde erstellt: {dateTime}"
    ],
    "action": {
      "download": "Backup jetzt herunterladen",
      "terminate": "Beenden"
    },
    "alert": {
      "download": "Das Backup l??uft jetzt und wird verschl??sselt und auf Ihren Computer geladen. Das wird eine Weile dauern. Sobald der Download begonnen hat, k??nnen Sie diese Seite verlassen."
    }
  },
  "es": {
    "getHelp": {
      "full": "Si tiene problemas, pruebe el {forum}.",
      "forum": "foro de la comunidad"
    },
    "notConfigured": [
      "Las copias de seguridad no est??n configuradas."
    ],
    "neverRun": [
      "La copia de seguridad configurada a??n no se ha ejecutado.",
      "Si ha configurado copias de seguridad en los ??ltimos d??as, esto es normal. De lo contrario, algo ha salido mal."
    ],
    "somethingWentWrong": [
      "??Algo est?? mal!",
      {
        "full": "La ??ltima copia de seguridad que se complet?? correctamente fue {moreThanThreeDaysAgo}.",
        "moreThanThreeDaysAgo": "hace m??s de tres d??as"
      }
    ],
    "success": [
      "La copia de seguridad est?? funcionando",
      "??ltima copia de seguridad completada con ??xito {dateTime}"
    ],
    "action": {
      "download": "Descargar copia de seguridad ahora",
      "terminate": "Terminar"
    },
    "alert": {
      "download": "La copia de seguridad se est?? ejecutando ahora y se cifrar?? y descargar?? en su computadora. Esto puede tardar un rato. Una vez que comience la descarga, puede salir de esta p??gina"
    }
  },
  "fr": {
    "getHelp": {
      "full": "Si vous rencontrez des difficult??s, merci de consulter le {forum}.",
      "forum": "forum de la communaut??"
    },
    "notConfigured": [
      "Sauvegarde non configur??",
      "Dans les versions pr??c??dentes d'ODK Central, il ??tait possible de mettre en place des sauvegardes automatiques sur Google Drive. Cette fonctionnalit?? est d??pr??ci??e, mais il y a d'autres moyens de sauvegarder les donn??es du serveur. Nous recommandons fortement ?? tous les utilisateurs de sauvegarder leurs donn??es."
    ],
    "neverRun": [
      "La sauvegarde configur??e n'a pas encore ??t?? lanc??e.",
      "Si vous avez configur?? des sauvegardes au cours des deux derniers jours, c'est normal. Sinon, quelque-chose s'est mal pass??.",
      "Dans ce cas, la solution la plus probable consiste ?? red??marrer le service."
    ],
    "somethingWentWrong": [
      "Quelque-chose ne va pas.",
      {
        "full": "La derni??re sauvegarde correctement termin??e ??tait {moreThanThreeDaysAgo}.",
        "moreThanThreeDaysAgo": "plus de trois jours auparavant."
      },
      "La solution la plus probable consiste ?? red??marrer le service."
    ],
    "success": [
      "La sauvegarde fonctionne.",
      "La derni??re sauvegarde correctement ex??cut??e {dateTime}."
    ],
    "action": {
      "download": "T??l??charger la sauvegarde maintenant",
      "terminate": "Terminer"
    },
    "alert": {
      "download": "La sauvegarde est en cours, elle sera chiffr??e et t??l??charg??e sur votre ordinateur. Cela peut prendre un peu de temps. Une fois d??marr?? le t??l??chargement, vous pourrez quitter cette page."
    }
  },
  "id": {
    "getHelp": {
      "full": "Apabila Anda memiliki kesulitan, silakan kunjungi {forum}.",
      "forum": "Forum komunitas"
    },
    "notConfigured": [
      "Data cadangan tidak terkonfigurasi."
    ],
    "neverRun": [
      "Data candangan yang dikonfigurasi belum berjalan.",
      "Hal ini adalah normal ppabila Anda memiliki data cadangan yang telah dikonfigurasi dalam beberapa hari terakhir. Jika tidak, terjadi sebuah kesalahan."
    ],
    "somethingWentWrong": [
      "Terjadi kesalahan!",
      {
        "full": "Data cadangan terakhir yang sukses dibuat adalah {moreThanThreeDaysAgo}.",
        "moreThanThreeDaysAgo": "lebih dari tiga hari yang lalu"
      }
    ],
    "success": [
      "Data cadangan telah berfungsi.",
      "Data cadangan terakhir telah sukses dibuat {dateTime}."
    ],
    "action": {
      "download": "Unduh cadangan sekarang",
      "terminate": "Putuskan"
    },
    "alert": {
      "download": "Pencadangan sedang berlangsung dan akan dienkripsi selanjutnya diunduh ke komputermu. Perlu waktu beberapa saat. Saat pengunduhan dimulai, kamu bisa meninggalkan laman ini."
    }
  },
  "it": {
    "getHelp": {
      "full": "Se hai problemi, consulta il {forum}.",
      "forum": "forum comunitario"
    },
    "notConfigured": [
      "I backup non sono configurati.",
      "Nelle versioni precedenti di ODK Central era possibile impostare backup automatici su Google Drive. Questa funzionalit?? ?? stata deprecata, ma esistono altri modi per eseguire il backup dei dati sul server. Consigliamo vivamente a tutti gli utenti di eseguire il backup dei propri dati."
    ],
    "neverRun": [
      "Il backup configurato non ?? ancora stato eseguito.",
      "Se hai configurato i backup negli ultimi due giorni, ?? normale. Altrimenti qualcosa ?? andato storto.",
      "In tal caso, la soluzione pi?? probabile ?? riavviare il servizio."
    ],
    "somethingWentWrong": [
      "Qualcosa non funziona!",
      {
        "full": "L'ultimo backup completato con successo ?? stato {moreThanThreeDaysAgo}.",
        "moreThanThreeDaysAgo": "pi?? di tre giorni fa"
      },
      "La soluzione pi?? probabile ?? riavviare il servizio."
    ],
    "success": [
      "Il Backup sta funzionando.",
      "L'ultimo backup ?? stato completato con successo {dateTime}."
    ],
    "action": {
      "download": "Scarica il backup ora",
      "terminate": "Terminare"
    },
    "alert": {
      "download": "Il backup ?? ora in esecuzione e verr?? crittografato e scaricato sul tuo computer. L'operazione potrebbe richiedere del tempo. Una volta avviato il download, puoi lasciare questa pagina."
    }
  },
  "ja": {
    "getHelp": {
      "full": "????????????????????????{forum}???????????????????????????",
      "forum": "????????????????????????????????????"
    },
    "notConfigured": [
      "???????????????????????????????????????????????????"
    ],
    "neverRun": [
      "??????????????????????????????????????????????????????",
      "???????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????"
    ],
    "somethingWentWrong": [
      "??????????????????????????????",
      {
        "full": "?????????????????????????????????{moreThanThreeDaysAgo}????????????????????????????????????",
        "moreThanThreeDaysAgo": "3????????????"
      }
    ],
    "success": [
      "????????????????????????????????????",
      "{dateTime}?????????????????????????????????????????????????????????"
    ],
    "action": {
      "download": "????????????????????????????????????????????????",
      "terminate": "??????"
    },
    "alert": {
      "download": "??????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????"
    }
  },
  "sw": {
    "getHelp": {
      "full": "Ikiwa unatatizika, tafadhali jaribu {forum}.",
      "forum": "jamii forum"
    },
    "notConfigured": [
      "Hifadhi rudufu hazijasanidiwa."
    ],
    "neverRun": [
      "Hifadhi rudufu iliyosanidiwa bado haijatekelezwa",
      "Ikiwa umesanidi nakala rudufu ndani ya siku chache zilizopita, hii ni kawaida. Vinginevyo, kitu kimeenda vibaya."
    ],
    "somethingWentWrong": [
      "Kuna kitu hakiko sawa",
      {
        "full": "Hifadhi rudufu ya hivi punde iliyokamilika kwa ufanisi ilikuwa {moreThanThreeDaysAgo}.",
        "moreThanThreeDaysAgo": "zaidi ya siku tatu zilizopita"
      }
    ],
    "success": [
      "Hifadhi rudufu inafanya kazi.",
      "Hifadhi rudufu ya mwisho ilikamilika {dateTime}."
    ],
    "action": {
      "download": "Pakua nakala rudufu sasa",
      "terminate": "Sitisha"
    },
    "alert": {
      "download": "Hifadhi rudufu inaendelea sasa, na itasimbwa kwa njia fiche na kupakuliwa kwenye kompyuta yako. Hii inaweza kuchukua muda. Mara tu upakuaji unapoanza, unaweza kuondoka kwenye ukurasa huu."
    }
  }
}
</i18n>
