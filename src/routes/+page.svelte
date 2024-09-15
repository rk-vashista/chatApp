<script>
	import { onMount } from 'svelte';
	import { auth } from '$lib/firebase';
	import {
		createUserWithEmailAndPassword,
		signInWithEmailAndPassword,
		sendPasswordResetEmail
	} from 'firebase/auth';
	import { goto } from '$app/navigation';
	import { LogIn, UserPlus, ArrowLeft, Mail, Lock } from 'lucide-svelte';
	import { fade, fly } from 'svelte/transition';

	let email = '';
	let password = '';
	let error = '';
	let successMessage = '';
	let isSignUp = false;
	let isResetPassword = false;

	async function handleEmailAuth() {
		try {
			if (isSignUp) {
				await createUserWithEmailAndPassword(auth, email, password);
			} else {
				await signInWithEmailAndPassword(auth, email, password);
			}
			goto('/chat');
		} catch (err) {
			error = err.message;
		}
	}

	async function handleResetPassword() {
		try {
			await sendPasswordResetEmail(auth, email);
			successMessage = 'Password reset email sent. Please check your inbox.';
			error = '';
		} catch (err) {
			error = err.message;
			successMessage = '';
		}
	}

	function resetForm() {
		email = '';
		password = '';
		error = '';
		successMessage = '';
	}

	onMount(() => {
		document.documentElement.classList.add('dark');
	});
</script>

<div class="min-h-screen bg-gradient-to-br from-gray-900 to-gray-800 text-gray-100 flex items-center justify-center p-4">
	<div class="w-full max-w-md space-y-8 bg-gray-800 p-8 rounded-xl shadow-2xl" in:fade={{ duration: 300, delay: 300 }}>
		<div class="text-center">
			<h2 class="mt-6 text-4xl font-bold text-blue-400">LLAMA Chat</h2>
			<p class="mt-2 text-sm text-gray-400">
				{#if isResetPassword}
					Reset your password
				{:else if isSignUp}
					Create an account to get started
				{:else}
					Sign in to your account
				{/if}
			</p>
		</div>
		<form
			class="mt-8 space-y-6"
			on:submit|preventDefault={isResetPassword ? handleResetPassword : handleEmailAuth}
		>
			<div class="rounded-md shadow-sm -space-y-px">
				<div>
					<label for="email" class="sr-only">Email address</label>
					<div class="relative">
						<Mail class="absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400" size={20} />
						<input
							id="email"
							name="email"
							type="email"
							bind:value={email}
							required
							class="appearance-none rounded-t-md relative block w-full px-3 py-2 border border-gray-700 placeholder-gray-500 text-gray-100 bg-gray-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500 focus:z-10 sm:text-sm pl-10 transition-colors duration-200"
							placeholder="Email address"
						/>
					</div>
				</div>
				{#if !isResetPassword}
					<div>
						<label for="password" class="sr-only">Password</label>
						<div class="relative">
							<Lock class="absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400" size={20} />
							<input
								id="password"
								name="password"
								type="password"
								bind:value={password}
								required
								class="appearance-none rounded-b-md relative block w-full px-3 py-2 border border-gray-700 placeholder-gray-500 text-gray-100 bg-gray-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500 focus:z-10 sm:text-sm pl-10 transition-colors duration-200"
								placeholder="Password"
							/>
						</div>
					</div>
				{/if}
			</div>

			<div>
				<button
					type="submit"
					class="group relative w-full flex justify-center py-2 px-4 border border-transparent text-sm font-medium rounded-md text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 transition-all duration-200 transform hover:scale-105"
				>
					<span class="absolute left-0 inset-y-0 flex items-center pl-3">
						{#if isResetPassword}
							<Mail class="h-5 w-5 text-blue-500 group-hover:text-blue-400" aria-hidden="true" />
						{:else if isSignUp}
							<UserPlus class="h-5 w-5 text-blue-500 group-hover:text-blue-400" aria-hidden="true" />
						{:else}
							<LogIn class="h-5 w-5 text-blue-500 group-hover:text-blue-400" aria-hidden="true" />
						{/if}
					</span>
					{#if isResetPassword}
						Reset Password
					{:else if isSignUp}
						Sign Up
					{:else}
						Sign In
					{/if}
				</button>
			</div>
		</form>
		<div class="text-center space-y-2">
			{#if !isResetPassword}
				<button
					on:click={() => {
						isSignUp = !isSignUp;
						resetForm();
					}}
					class="text-sm text-blue-400 hover:text-blue-300 focus:outline-none focus:underline transition-colors duration-200"
				>
					{isSignUp ? 'Already have an account? Sign In' : "Don't have an account? Sign Up"}
				</button>
				<button
					on:click={() => {
						isResetPassword = true;
						resetForm();
					}}
					class="block w-full text-sm text-blue-400 hover:text-blue-300 focus:outline-none focus:underline transition-colors duration-200"
				>
					Forgot your password?
				</button>
			{:else}
				<button
					on:click={() => {
						isResetPassword = false;
						resetForm();
					}}
					class="text-sm text-blue-400 hover:text-blue-300 focus:outline-none focus:underline transition-colors duration-200 flex items-center justify-center"
				>
					<ArrowLeft class="h-4 w-4 mr-1" />
					Back to Sign In
				</button>
			{/if}
		</div>
		{#if error}
			<div
				class="mt-4 text-center text-sm text-red-400 bg-red-900 p-2 rounded-md"
				in:fly={{ y: 20, duration: 300 }}
			>
				{error}
			</div>
		{/if}
		{#if successMessage}
			<div
				class="mt-4 text-center text-sm text-green-400 bg-green-900 p-2 rounded-md"
				in:fly={{ y: 20, duration: 300 }}
			>
				{successMessage}
			</div>
		{/if}
	</div>
</div>

<style>
	:global(body) {
		background-color: #1a202c;
		color: #f7fafc;
	}
</style>