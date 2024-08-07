<script>
  import {
    questionnaire,
    deleteConfirmation,
    questionnaireValidation,
    handleAddOption,
    handleRemoveOption,
    handleRemoveQuestion,
    handleCode,
    handleAnswerSelect,
    addDynamicValue
  } from '../store/exercise';
  import { Select, SelectItem } from 'carbon-components-svelte';

  import TrashCanIcon from 'carbon-icons-svelte/lib/TrashCan.svelte';
  import AddFilledIcon from 'carbon-icons-svelte/lib/AddFilled.svelte';
  import CheckmarkFilledIcon from 'carbon-icons-svelte/lib/CheckmarkFilled.svelte';
  import CheckmarkOutlineIcon from 'carbon-icons-svelte/lib/CheckmarkOutline.svelte';

  import TextField from '$lib/components/Form/TextField.svelte';
  import Modal from '$lib/components/Modal/index.svelte';
  import TextArea from '$lib/components/Form/TextArea.svelte';
  import Checkbox from '$lib/components/Form/Checkbox.svelte';
  import RadioItem from '$lib/components/Form/RadioItem.svelte';
  import IconButton from '$lib/components/IconButton/index.svelte';
  import ErrorMessage from '$lib/components/ErrorMessage/index.svelte';
  import PrimaryButton from '$lib/components/PrimaryButton/index.svelte';
  import { VARIANTS } from '$lib/components/PrimaryButton/constants';
  import QuestionContainer from '$lib/components/QuestionContainer/index.svelte';
  import DeleteConfirmationModal from './DeleteConfirmation.svelte';
  import OrderModal from './OrderModal.svelte';
  import { QUESTION_TYPE, QUESTION_TYPES } from '$lib/components/Question/constants';
  import { filterOutDeleted } from './functions';
  import { deleteExercise } from '$lib/utils/services/courses';
  import { lesson } from '../store/lessons';
  import { t } from '$lib/utils/functions/translations';

  const initialQuestionsLength = $questionnaire.questions.length;

  export let shouldDeleteExercise = false;
  export let exerciseId;
  export let goBack = () => {};

  let errors = {};
  let questionIdToDelete = null;
  let questions = [];
  let isDeleting = false;

  function shouldScrollToLast(questionId, questions) {
    const [lastQuestion] = questions.slice(-1);
    const currentQuestionsLength = questions.length;
    const isLast = lastQuestion.id === questionId;

    return isLast && initialQuestionsLength !== currentQuestionsLength;
  }

  function onInitDeleteClicked(questionId) {
    return () => {
      questionIdToDelete = questionId;
      $deleteConfirmation.open = true;
    };
  }

  function onFinalDeleteClicked() {
    handleRemoveQuestion(questionIdToDelete)();
  }

  function getQuestionErrorMsg(errors, question, errorKey) {
    return errors[question.id] ? errors[question.id][errorKey] : null;
  }

  async function handleDelete() {
    console.log('delete');
    isDeleting = true;
    await deleteExercise(questions, exerciseId);

    lesson.update((_lesson) => ({
      ..._lesson,
      exercises: _lesson.exercises.filter((exercise) => exercise.id !== exerciseId)
    }));
    goBack();
  }

  $: errors = $questionnaireValidation;
  $: questions = filterOutDeleted($questionnaire.questions);
</script>

<DeleteConfirmationModal
  onCancel={() => (questionIdToDelete = null)}
  onDelete={onFinalDeleteClicked}
/>

<OrderModal />

<Modal
  onClose={() => (shouldDeleteExercise = false)}
  bind:open={shouldDeleteExercise}
  width="w-2/4"
  modalHeading={$t('course.navItem.lessons.exercises.all_exercises.edit_mode.delete_modal')}
>
  <form on:submit|preventDefault>
    <h1 class="dark:text-white text-xl">
      {$t('course.navItem.lessons.exercises.all_exercises.edit_mode.sure')}
    </h1>

    <div class="mt-5 flex items-center justify-between">
      <PrimaryButton
        className="px-6 py-3"
        variant={VARIANTS.OUTLINED}
        label={$t('course.navItem.lessons.exercises.all_exercises.edit_mode.no')}
        type="submit"
        onClick={() => (shouldDeleteExercise = false)}
      />
      <PrimaryButton
        className="px-6 py-3"
        variant={VARIANTS.CONTAINED}
        label={isDeleting
          ? $t('course.navItem.lessons.exercises.all_exercises.edit_mode.deleting')
          : $t('course.navItem.lessons.exercises.all_exercises.edit_mode.yes')}
        isDisabled={isDeleting}
        onClick={handleDelete}
      />
    </div>
  </form>
</Modal>

<div class="w-full mb-20">
  {#if Object.values(errors).length}
    <div class="w-full flex justify-center mb-4">
      <ErrorMessage
        message={$t('course.navItem.lessons.exercises.all_exercises.edit_mode.error')}
      />
    </div>
  {/if}
  <div class="questions pt-6 px-6">
    {#each questions as question (question.id)}
      <!-- {#key question.id} -->
      <QuestionContainer
        onClose={onInitDeleteClicked(question.id)}
        scrollToQuestion={shouldScrollToLast(question.id, $questionnaire.questions)}
        bind:points={question.points}
        hasError={!!errors[question.id]}
        onPointsChange={() => {
          question.is_dirty = true;
        }}
      >
        <div class="flex justify-between items-center">
          <div class="mr-5 w-3/5">
            <TextField
              placeholder={$t('course.navItem.lessons.exercises.all_exercises.edit_mode.question')}
              bind:value={question.title}
              isRequired={true}
              onChange={() => {
                question.is_dirty = true;
              }}
            />
          </div>

          <Select
            size="xl"
            class="w-[50px]"
            bind:selected={question.question_type.id}
            on:change={(e) => {
              const id = parseInt(e.target?.value);
              question.question_type = QUESTION_TYPES.find((q) => q.id === id);
              question.is_dirty = true;
            }}
          >
            {#each QUESTION_TYPES as type}
              <SelectItem value={type.id} text={$t(type.label)} />
            {/each}
          </Select>
        </div>

        {#if typeof question.code === 'string'}
          <div class="flex justify-between items-center my-3 w-3/5">
            <TextArea
              bind:value={question.code}
              rows={2}
              placeholder={$t('course.navItem.lessons.exercises.all_exercises.edit_mode.write')}
            />
            <IconButton value="write-code" onClick={() => handleCode(question.id, false)}>
              <TrashCanIcon size={24} class="carbon-icon dark:text-white" />
            </IconButton>
          </div>
        {/if}

        <div class="flex flex-col mt-2">
          {#if QUESTION_TYPE.RADIO === question.question_type.id}
            {#each filterOutDeleted(question.options) as option}
              <RadioItem
                isEditable={true}
                name={question.title || 'radio-name'}
                bind:label={option.label}
                onChange={addDynamicValue(question.id, option.id)}
              >
                <div slot="iconbutton" class="flex items-center">
                  <IconButton
                    value={option.id}
                    onClick={handleRemoveOption(question.id, option.id)}
                  >
                    <TrashCanIcon size={24} class="carbon-icon dark:text-white" />
                  </IconButton>
                  <IconButton
                    value={option.id}
                    onClick={handleAnswerSelect(question.id, option.id)}
                    buttonClassName={option.is_correct && 'success'}
                  >
                    {#if option.is_correct}
                      <CheckmarkFilledIcon size={24} class="carbon-icon dark:text-white" />
                    {:else}
                      <CheckmarkOutlineIcon size={24} class="carbon-icon dark:text-white" />
                    {/if}
                  </IconButton>
                </div>
              </RadioItem>
            {/each}
          {/if}

          {#if QUESTION_TYPE.CHECKBOX === question.question_type.id}
            {#each filterOutDeleted(question.options) as option}
              <Checkbox
                isEditable={true}
                name={question || 'checkbox-name'}
                bind:label={option.label}
                onChange={addDynamicValue(question.id, option.id)}
              >
                <div slot="iconbutton" class="flex items-center">
                  <IconButton
                    value={option.id}
                    onClick={handleRemoveOption(question.id, option.id)}
                  >
                    <TrashCanIcon size={24} class="carbon-icon dark:text-white" />
                  </IconButton>
                  <IconButton
                    value={option.id}
                    onClick={handleAnswerSelect(question.id, option.id)}
                    buttonClassName={option.is_correct && 'success'}
                  >
                    {#if option.is_correct}
                      <CheckmarkFilledIcon size={24} class="carbon-icon dark:text-white" />
                    {:else}
                      <CheckmarkOutlineIcon size={24} class="carbon-icon dark:text-white" />
                    {/if}
                  </IconButton>
                </div>
              </Checkbox>
            {/each}
          {/if}

          {#if QUESTION_TYPE.TEXTAREA === question.question_type.id}
            <TextArea bind:value={question.value} disabled={true} />
          {/if}

          {#if getQuestionErrorMsg(errors, question, 'option')}
            <ErrorMessage message={getQuestionErrorMsg(errors, question, 'option')} />
          {/if}
        </div>

        {#if QUESTION_TYPE.TEXTAREA !== question.question_type.id}
          <div class="flex items-center mt-3">
            <PrimaryButton
              disablePadding={true}
              className="p-2"
              variant={VARIANTS.OUTLINED}
              onClick={handleAddOption(question.id)}
            >
              <AddFilledIcon size={24} class="carbon-icon dark:text-white" />
              <p class="dark:text-white ml-2">
                {$t('course.navItem.lessons.exercises.all_exercises.edit_mode.option')}
              </p>
            </PrimaryButton>
          </div>
        {/if}
      </QuestionContainer>

      <!--  {/key} -->
    {/each}
  </div>
</div>

<style>
  :global(#dnd-action-dragged-el div) {
    z-index: 100;
  }

  :global(select) {
    background-image: none;
  }
</style>
