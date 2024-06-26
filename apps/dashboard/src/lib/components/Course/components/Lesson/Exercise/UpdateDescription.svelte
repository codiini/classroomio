<script lang="ts">
  import QuestionContainer from '$lib/components/QuestionContainer/index.svelte';
  import DateTime from '$lib/components/Form/DateTime.svelte';
  import TextField from '$lib/components/Form/TextField.svelte';
  import { questionnaire } from '../store/exercise';
  import TextEditor from '$lib/components/TextEditor/index.svelte';
  import { t } from '$lib/utils/functions/translations';

  export let preview: boolean;

  function getTotalPossibleGrade(questions: { points: string }[]) {
    return questions.reduce((acc, question) => {
      acc += parseFloat(question.points, 10);
      return acc;
    }, 0);
  }
</script>

<div class="mb-5 {!preview ? 'px-6' : 'px-2'}">
  <QuestionContainer isTitle={true}>
    {#if !preview}
      <TextField
        placeholder={$t('course.navItem.lessons.exercises.all_exercises.description.title')}
        bind:value={$questionnaire.title}
        className="mb-2"
        onChange={() => ($questionnaire.is_title_dirty = true)}
      />
      <DateTime
        label={$t('course.navItem.lessons.exercises.all_exercises.view_mode.due')}
        className="w-50"
        value={$questionnaire.due_by}
        onInput={(e) => {
          $questionnaire.due_by = e.target.value;
          $questionnaire.is_due_by_dirty = true;
        }}
      />

      <div class="mt-3">
        <p class="mb-1">
          {$t('course.navItem.lessons.exercises.all_exercises.description.heading')}
        </p>
        <TextEditor
          value={$questionnaire.description}
          onChange={(html) => {
            $questionnaire.is_description_dirty = true;
            $questionnaire.description = html;
          }}
          placeholder={$t('course.navItem.lessons.exercises.all_exercises.description.describe')}
          maxHeight={300}
        />
      </div>
    {:else if preview}
      <h2 class="my-1">{$questionnaire.title}</h2>
      <div class="flex items-center">
        <p class="dark:text-white mx-2">
          <strong>{$questionnaire.questions.length}</strong>
          {$t('course.navItem.lessons.exercises.all_exercises.view_mode.questions')}
        </p>
        |
        <p class="dark:text-white mx-2">
          <strong>{getTotalPossibleGrade($questionnaire.questions)}</strong>
          {$t('course.navItem.lessons.exercises.all_exercises.view_mode.points')}.
        </p>
        |
        <p class="dark:text-white mx-2">
          {$t('course.navItem.lessons.exercises.all_exercises.view_mode.all')}
        </p>
        {#if $questionnaire.due_by}
          |
          <p class="dark:text-white mx-2">
            <strong>{$t('course.navItem.lessons.exercises.all_exercises.view_mode.due')}:</strong>
            {new Date($questionnaire.due_by).toLocaleString()}
          </p>
        {/if}
      </div>

      <article class="mt-3 preview prose prose-sm sm:prose p-2">
        {@html $questionnaire.description ||
          $t('course.navItem.lessons.exercises.all_exercises.description.no')}
      </article>
    {/if}
  </QuestionContainer>
</div>
