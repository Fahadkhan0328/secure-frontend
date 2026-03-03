<script lang="ts">
  import { SignedOut, SignInButton, SignedIn } from 'svelte-clerk';
</script>

<div class="text-center mt-[-10vh]">

  <h2 class="text-6xl md:text-7xl font-extrabold text-gray-900 tracking-tight mb-8">
    Welcome to your <br />
    <span class="text-transparent bg-clip-text bg-gradient-to-r from-indigo-600 via-purple-600 to-pink-500">
      Secure Architecture
    </span>
  </h2>
  
  <p class="mt-4 text-xl md:text-2xl text-gray-600 max-w-3xl mx-auto leading-relaxed mb-12">
    The frontend Bouncer is active. Log in to retrieve your JWT and unlock access to the high-performance Python backend.
  </p>

  <div class="flex justify-center">
    
    <SignedOut>
      <SignInButton mode="modal">
        <button class="relative inline-flex items-center justify-center px-10 py-4 text-lg font-bold text-white transition-all duration-200 bg-indigo-600 border border-transparent rounded-full shadow-xl hover:bg-indigo-700 hover:shadow-2xl hover:-translate-y-1 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-600">
          Sign In to Continue
          <svg class="w-5 h-5 ml-3 -mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z"></path></svg>
        </button>
      </SignInButton>
    </SignedOut>

    <SignedIn>
      <a href="/dashboard" class="relative inline-flex items-center justify-center px-10 py-4 text-lg font-bold text-white transition-all duration-200 bg-gradient-to-r from-purple-600 to-indigo-600 border border-transparent rounded-full shadow-xl hover:from-purple-700 hover:to-indigo-700 hover:shadow-2xl hover:-translate-y-1 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-600">
        Enter Dashboard
        <svg class="w-5 h-5 ml-3 -mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"></path></svg>
      </a>
    </SignedIn>

  </div>
</div>