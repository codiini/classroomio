<script>
  import { course } from '$lib/components/Course/store';
  import { currentOrg } from '$lib/utils/store/org';

  export let studentName = '';
  const logo = '/logo-512.png';

  const dateIssued = new Date().toLocaleDateString();
</script>

<div class="p-3 w-full dark:text-black bg-[#95449A]">
  <div class="w-full bg-white pb-5 flex">
    <div class="w-[80%] pl-3">
      <header class="flex justify-start items-center gap-1 text-sm font-bold">
        <img src={$currentOrg.avatar_url ? $currentOrg.avatar_url : logo} alt="logo" class="w-5" />
        <h1 class="text-sm">{$currentOrg.name}</h1>
      </header>

      <div class="font-bold mt-7">
        <h2 class="text-[7px] m-0">This certificate is awarded to</h2>
        <h1 class="text-lg m-0 mt-0.5 leading-3 text-[#95449A]">{studentName}</h1>
      </div>

      <div class="font-bold mt-3 w-[90%]">
        <h1 class="text-[7px]">has succesfully completed training on</h1>
        <h2 class="text-lg m-0">
          {$course.title}
        </h2>
      </div>

      <div class="flex justify-between items-center mt-8 mb-5 text-center">
        <div class="">
          <h1 class="h-2/4 m-0 text-[7px]">DATE ISSUED:</h1>
          <p class="border-b border-black font-bold text-[7px]">{dateIssued}</p>
        </div>
        <div class="w-[20%] text-[7px]">
          <div class=""></div>
          <p class="border-t border-black font-bold text-[7px]">Signature</p>
        </div>
      </div>

      <footer class="text-[8px] font-bold w-[90%] desc">
        {$course.description}
      </footer>
    </div>

    <div class="w-[20%]">
      <img src="/images/purple-professional-badge.svg" alt="A badge" class="w-full" />
    </div>
  </div>
</div>
