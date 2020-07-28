<script>
import { onMount } from 'svelte';
import { fetchMovies } from '../api';
//config
import {IMAGE_BASE_URL, BACKDROP_SIZE, POSTER_SIZE} from '../config.js';
//components
import Header from '../components/Header.svelte';
import Hero from '../components/Hero.svelte';
import Search from '../components/Search.svelte';
import Grid from '../components/Grid.svelte';
import Thumb from '../components/Thumb.svelte';
import LoadMoreButton from '../components/LoadMoreButton.svelte';
import Spinner from '../components/Spinner.svelte';

let movies = {movies:[]};
let isLoading;
let searchTerm ='';
let error;

const handleFetchMovies = async (loadMore, searchTerm)=>{
    try {
        isLoading = true;
        error = false;
        movies = await fetchMovies (movies, loadMore, searchTerm);
    } catch (err) {
        error = true;
    }
    isLoading = false;
}

onMount(async ()=>{
    const sessionStorage = window.sessionStorage.getItem('svelte-movie');
    if(sessionStorage){
        console.log('Grabbing from sessionStorage.');
    }else{
        console.log('Grabbing from API.');
        handleFetchMovies(false, searchTerm);
    }
   
});

const handleSearch = event => {
    searchTerm = event.detail.searchText;
    movies.movies = [];
    handleFetchMovies(false, searchTerm)
}

const handleLoadMore = ()=> handleFetchMovies(true, searchTerm);

$: {
    if(movies.movies.length > 0){
        window.sessionStorage.setItem('svelte-movies',JSON.stringify(movies));
    }
}
</script>


{#if error}
<p>Something went wrong ...</p>
{:else}


{#if movies.heroImage && !searchTerm}
    
    <Hero
    image={`${IMAGE_BASE_URL}${BACKDROP_SIZE}${movies.heroImage.backdrop_path}`}
    title={movies.heroImage.original_title}
    text={movies.heroImage.overview}
    />
    {/if}
{/if}

<Search on:search={handleSearch } />
<Grid header={searchTerm ? 'Search Result' : 'Popular Movies'}>
    {#each movies.movies as movie}
        <Thumb
        clickable
        image={movie.poster_path && IMAGE_BASE_URL + POSTER_SIZE + movie.poster_path}
        movieId={movie.id}/>
    {/each}
</Grid>

{#if isLoading}
    <Spinner />
{/if}

{#if !isLoading && movies.currentPage<movies.totalPages}
    <LoadMoreButton on:loadMore={handleLoadMore}>Load More</LoadMoreButton>
{/if}

<style>

</style>
