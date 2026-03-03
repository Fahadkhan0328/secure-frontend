<script lang="ts">
  import '../app.css';
  import { ClerkProvider, SignedIn, UserButton } from 'svelte-clerk';
</script>

<ClerkProvider>
  <div class="min-h-screen bg-gradient-to-br from-indigo-50 via-white to-purple-50 text-gray-900 font-sans">
    
    <nav class="absolute top-0 w-full p-6 pt-12">
      <div class="max-w-7xl mx-auto flex justify-between items-center relative">
        <div class="w-10"></div> 
        
        <div class="absolute left-1/2 transform -translate-x-1/2">
          <h1 class="text-6xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-indigo-600 to-purple-600 tracking-tighter">
            SecureApp
          </h1>
        </div>
        
        <div class="z-10 relative">
          <SignedIn>
            <UserButton />
          </SignedIn>
        </div>
      </div>
    </nav>

    <main class="flex flex-col justify-center min-h-screen max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 relative z-0">
      <slot />
    </main>
    
  </div>
</ClerkProvider>