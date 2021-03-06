<template>
  <form :class="$style.editEvent" @submit.prevent="onSubmit">
    <vue-grid>
      <vue-grid-row>
        <vue-grid-item>
          <h1 v-if="event.id === null">{{ $t('common.add.event' /* Add a new Event */) }}</h1>
          <h1 v-if="event.id !== null">{{ $t('common.edit.event' /* Edit Event */) }}</h1>


        </vue-grid-item>
      </vue-grid-row>

      <vue-grid-row>
        <vue-grid-item>
          <vue-input
            name="title"
            id="title"
            required
            :placeholder="$t('common.title' /* Title */)"
            validation="required"
            v-model="event.title" />
        </vue-grid-item>
      </vue-grid-row>

      <vue-grid-row>
        <vue-grid-item>
          <vue-date-picker
            required
            :min-date="new Date('1950-01-01')"
            :current-date="event.date ? new Date(event.date) : new Date()"
            :first-day-of-week="1"
            :placeholder="$t('common.date' /* Date */)"
            @change="event.date = $event.toUTCString()" />
        </vue-grid-item>
      </vue-grid-row>

      <vue-grid-row>
        <vue-grid-item>
          <label>{{ $t('common.storyline' /* Storyline */) }}</label>
          <vue-select
            required
            name="storyline"
            id="storyline"
            :value="event.storyline.toString()"
            :options="storylines"
            @input="event.storyline = parseInt($event, 10)" />
        </vue-grid-item>
      </vue-grid-row>

      <vue-grid-row>
        <vue-grid-item>
          <label>{{ $t('common.chapter' /* Chapter */) }}</label>
          <vue-select
            required
            name="chapter"
            id="chapter"
            :value="event.chapter.toString()"
            :options="chapters"
            @input="event.chapter = parseInt($event, 10)" />
        </vue-grid-item>
      </vue-grid-row>

      <vue-grid-row>
        <vue-grid-item>
          <label>{{ $t('common.storylineOrder' /* Order */) }}</label>
          <vue-select
            required
            name="storylineOrder"
            id="storylineOrder"
            :value="event.storylineOrder.toString()"
            :options="storylineOrders"
            @input="event.storylineOrder = parseInt($event, 10)" />
        </vue-grid-item>
      </vue-grid-row>

      <vue-grid-row>
        <vue-grid-item>
          <vue-autocomplete
            :options="characters"
            :placeholder="$t('common.search.characters' /* Search for Characters */)"
            @request="getCharacters"
            @change="event.characters.push($event.value)" />
        </vue-grid-item>
      </vue-grid-row>

      <vue-grid-row>
        <vue-grid-item>
          <h5>{{ $t('common.characters' /* Characters */) }}</h5>
          <ul :class="$style.characterList">
            <li v-for="character in eventCharacters">
              {{ character.name }}
              <vue-button
                warn
                icon="trash"
                :aria-label="$t('common.delete' /* Delete */)"
                @click.prevent="removeCharacter(character)" />
            </li>
          </ul>
        </vue-grid-item>
      </vue-grid-row>

      <vue-grid-row>
        <vue-grid-item>
          <vue-textarea
            name="notes"
            id="notes"
            :placeholder="$t('common.notes' /* Notes */)"
            v-model="event.notes" />
        </vue-grid-item>
      </vue-grid-row>

      <vue-grid-row>
        <vue-grid-item>
          <vue-button accent>
            {{ $t('common.save' /* Save */) }}
          </vue-button>

          <vue-button
            @click="$router.push(`/story/${$route.params.storyId}`)">
            {{ $t('common.cancel' /* Cancel */) }}
          </vue-button>
        </vue-grid-item>
      </vue-grid-row>
    </vue-grid>
  </form>
</template>

<script lang="ts">
  import { mapActions, mapGetters }      from 'vuex';
  import VueGrid                         from '../../shared/components/VueGrid/VueGrid';
  import VueGridItem                     from '../../shared/components/VueGridItem/VueGridItem';
  import VueButton                       from '../../shared/components/VueButton/VueButton';
  import VueGridRow                      from '../../shared/components/VueGridRow/VueGridRow';
  import { IEvent }                      from '../IEvent';
  import VueInput                        from '../../shared/components/VueInput/VueInput';
  import VueSelect, { IVueSelectOption } from '../../shared/components/VueSelect/VueSelect';
  import { createGUID }                  from '../../shared/utils/misc';
  import { addNotification }             from '../../shared/components/VueNotificationStack/utils';
  import VueAutocomplete                 from '../../shared/components/VueAutocomplete/VueAutocomplete';
  import { ICharacter }                  from '../../character/ICharacter';
  import { IAutocompleteOption }         from '../../shared/components/VueAutocomplete/IAutocompleteOption';
  import VueDatePicker                   from '../../shared/components/VueDatePicker/VueDatePicker';
  import VueTextarea                     from '../../shared/components/VueTextarea/VueTextarea';

  interface IData {
    event: IEvent;
    characters: IAutocompleteOption[];
    chapters: IVueSelectOption[];
    storylineOrders: IVueSelectOption[];
    storylines: IVueSelectOption[];
  }

  export default {
    name:          'EditEvent',
    $_veeValidate: {
      validator: 'new',
    },
    metaInfo:      {
      title: 'Chaptr | Edit Event',
    },
    computed:      {
      ...mapGetters('character', ['allCharacters', 'getCharacterById']),
      ...mapGetters('event', ['getEventsByStoryId', 'getEventById']),
      eventCharacters() {
        const characters: ICharacter[] = [];

        this.event.characters.forEach((characterId: string) => {
          characters.push(this.getCharacterById(characterId));
        });

        return characters;
      },
    },
    components:    {
      VueTextarea,
      VueDatePicker,
      VueAutocomplete,
      VueSelect,
      VueInput,
      VueGrid,
      VueGridItem,
      VueButton,
      VueGridRow,
    },
    data(): IData {
      return {
        event:           {
          id:             null,
          storyId:        null,
          title:          '',
          characters:     [],
          chapter:        0,
          storylineOrder: -1,
          storyline:      0,
          date:           '',
          notes:          '',
        },
        characters:      [],
        chapters:        [],
        storylineOrders: [],
        storylines:      [],
      };
    },
    methods:       {
      ...mapActions('event', [
        'addEvent',
        'updateEvent',
      ]),
      onSubmit() {
        const storyId: string = this.$route.params.storyId;

        if (this.event.id === null) {
          this.event.id = createGUID();
          this.event.storyId = storyId;
          this.addEvent(this.event);
          addNotification({
                            title: this.$t('notification.event.add.title' /* Event added! */),
                            text:  this.$t('notification.event.add.text' /* Your Event has been saved! */),
                          });
        } else {
          this.updateEvent(this.event);
          addNotification({
                            title: this.$t('notification.event.update.title' /* Event updated! */),
                            text:  this.$t('notification.event.update.text' /* Your Event has been updated! */),
                          });
        }

        this.$router.push(`/story/${storyId}`);
      },
      getCharacters(query: string) {
        this.characters = this.allCharacters
                              .filter((character: ICharacter): boolean => {
                                return character.name.toLowerCase().indexOf(query.toLowerCase()) > -1
                                  && this.event.characters.indexOf(character.id) === -1;
                              })
                              .map((character: ICharacter): IAutocompleteOption => {
                                return {
                                  label: character.name,
                                  value: character.id,
                                };
                              });
      },
      fillChapters() {
        this.chapters.push({ label: this.$t('common.choose.chapter' /* Choose a Chapter */), value: '0' });
        for (let i = 1; i <= 99; i++) {
          this.chapters.push({ label: i.toString(), value: i.toString() });
        }
      },
      fillStoryLines() {
        this.storylines.push({ label: this.$t('common.choose.storyline' /* Choose a Storyline */), value: '0' });
        for (let i = 1; i <= 20; i++) {
          this.storylines.push({ label: i.toString(), value: i.toString() });
        }
      },
      fillStoryLineOrders(storyEvents: IEvent[]) {
        this.storylineOrders.push({
                                    label: this.$t('common.choose.storylineOrder' /* Choose Storyline Order */),
                                    value: '-1',
                                  });
        for (let i = 0; i < storyEvents.length; i++) {
          this.storylineOrders.push({ label: `${i + 1} - ${storyEvents[i].title}`, value: i.toString() });
        }

        this.storylineOrders.push({
                                    label: `${storyEvents.length + 1} - ${this.$t('common.event.new' /* New Event */)}`,
                                    value: storyEvents.length.toString(),
                                  });
        this.event.storylineOrder = this.event.storylineOrder === -1 ? storyEvents.length : this.event.storylineOrder;
      },
      removeCharacter(character: ICharacter) {
        this.event.characters = this.event.characters
                                    .filter((c: string) => {
                                      return c !== character.id;
                                    });
      },
    },
    mounted() {
      const storyEvents: IEvent[] = this.getEventsByStoryId(this.$route.params.storyId);

      if (this.$route.params.id) {
        this.event = this.getEventById(this.$route.params.id);
      }

      this.fillChapters();
      this.fillStoryLines();
      this.fillStoryLineOrders(storyEvents);
    },
  };
</script>


<style lang="scss" module>
  @import "../../shared/styles";

  .editEvent {
    h5 {
      margin-top: 0;
    }
  }

  .characterList {
    margin-bottom: $space-unit * 4;
    list-style:    initial;

    li {
      display: block;
      height:  $space-unit * 6;
    }

    button {
      min-width: auto;
      float:     right;
      padding:   0;
      height:    $space-unit * 4;
      width:     $space-unit * 4;
    }
  }
</style>
