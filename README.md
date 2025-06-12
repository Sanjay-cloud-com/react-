// File: src/main.jsx
import React from "react";
import ReactDOM from "react-dom/client";
import { BrowserRouter } from "react-router-dom";
import App from "./App";
import "./index.css";

ReactDOM.createRoot(document.getElementById("root")).render(
  <BrowserRouter>
    <App />
  </BrowserRouter>
);

// File: src/App.jsx
import React from "react";
import { Routes, Route } from "react-router-dom";
import SearchPage from "./pages/SearchPage";
import MovieDetails from "./pages/MovieDetails";

export default function App() {
  return (
    <Routes>
      <Route path="/" element={<SearchPage />} />
      <Route path="/movie/:id" element={<MovieDetails />} />
    </Routes>
  );
}

// File: src/services/omdbApi.js
const API_KEY = "your_api_key_here";
const BASE_URL = `https://www.omdbapi.com/?apikey=${API_KEY}`;

export const searchMovies = async (query, page = 1, type = "") => {
  const res = await fetch(`${BASE_URL}&s=${query}&page=${page}&type=${type}`);
  return await res.json();
};

export const getMovieDetails = async (id) => {
  const res = await fetch(`${BASE_URL}&i=${id}&plot=full`);
  return await res.json();
};

// File: src/components/SearchBar.jsx
import React from "react";

export default function SearchBar({ query, setQuery, onSearch }) {
  return (
    <div className="flex items-center gap-2">
      <input
        className="border p-2 rounded w-full"
        type="text"
        placeholder="Search movies..."
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      <button className="bg-blue-500 text-white p-2 rounded" onClick={onSearch}>
        Search
      </button>
    </div>
  );
}

// File: src/components/DropdownFilter.jsx
import React from "react";

export default function DropdownFilter({ type, setType }) {
  return (
    <select
      className="border p-2 rounded"
      value={type}
      onChange={(e) => setType(e.target.value)}
    >
      <option value="">All</option>
      <option value="movie">Movie</option>
      <option value="series">Series</option>
      <option value="episode">Episode</option>
    </select>
  );
}

// File: src/components/MovieCard.jsx
import React from "react";
import { Link } from "react-router-dom";

export default function MovieCard({ movie }) {
  return (
    <Link to={`/movie/${movie.imdbID}`} className="p-2 border rounded shadow">
      <img src={movie.Poster !== "N/A" ? movie.Poster : "https://via.placeholder.com/150"} alt={movie.Title} className="w-full h-64 object-cover" />
      <div className="mt-2 font-bold">{movie.Title}</div>
      <div>{movie.Year}</div>
    </Link>
  );
}

// File: src/components/MovieList.jsx
import React from "react";
import MovieCard from "./MovieCard";

export default function MovieList({ movies }) {
  return (
    <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
      {movies.map((movie) => (
        <MovieCard key={movie.imdbID} movie={movie} />
      ))}
    </div>
  );
}

// File: src/components/Pagination.jsx
import React from "react";

export default function Pagination({ currentPage, totalPages, onPageChange }) {
  return (
    <div className="flex justify-center gap-2 mt-4">
      {Array.from({ length: totalPages }, (_, i) => (
        <button
          key={i}
          className={`px-3 py-1 rounded ${currentPage === i + 1 ? "bg-blue-600 text-white" : "bg-gray-200"}`}
          onClick={() => onPageChange(i + 1)}
        >
          {i + 1}
        </button>
      ))}
    </div>
  );
}

// File: src/pages/SearchPage.jsx
import React, { useState, useEffect } from "react";
import { searchMovies } from "../services/omdbApi";
import SearchBar from "../components/SearchBar";
import DropdownFilter from "../components/DropdownFilter";
import MovieList from "../components/MovieList";
import Pagination from "../components/Pagination";

export default function SearchPage() {
  const [query, setQuery] = useState("Batman");
  const [type, setType] = useState("");
  const [movies, setMovies] = useState([]);
  const [page, setPage] = useState(1);
  const [totalResults, setTotalResults] = useState(0);
  const [error, setError] = useState("");

  const fetchMovies = async () => {
    try {
      const data = await searchMovies(query, page, type);
      if (data.Response === "True") {
        setMovies(data.Search);
        setTotalResults(parseInt(data.totalResults));
        setError("");
      } else {
        setMovies([]);
        setTotalResults(0);
        setError(data.Error);
      }
    } catch (err) {
      setError("Failed to fetch movies. Try again later.");
    }
  };

  useEffect(() => {
    fetchMovies();
  }, [page, type]);

  return (
    <div className="p-4 max-w-7xl mx-auto">
      <div className="flex justify-between items-center gap-4 mb-4">
        <SearchBar query={query} setQuery={setQuery} onSearch={() => fetchMovies()} />
        <DropdownFilter type={type} setType={setType} />
      </div>

      {error && <div className="text-red-500 mb-4">{error}</div>}

      <MovieList movies={movies} />
      {totalResults > 10 && (
        <Pagination
          currentPage={page}
          totalPages={Math.ceil(totalResults / 10)}
          onPageChange={setPage}
        />
      )}
    </div>
  );
}

// File: src/pages/MovieDetails.jsx
import React, { useEffect, useState } from "react";
import { useParams, Link } from "react-router-dom";
import { getMovieDetails } from "../services/omdbApi";

export default function MovieDetails() {
  const { id } = useParams();
  const [movie, setMovie] = useState(null);
  const [error, setError] = useState("");

  useEffect(() => {
    const fetchDetails = async () => {
      const data = await getMovieDetails(id);
      if (data.Response === "True") {
        setMovie(data);
      } else {
        setError(data.Error);
      }
    };
    fetchDetails();
  }, [id]);

  if (error) return <div className="p-4 text-red-500">{error}</div>;
  if (!movie) return <div className="p-4">Loading...</div>;

  return (
    <div className="p-4 max-w-4xl mx-auto">
      <Link to="/" className="text-blue-500 underline mb-4 inline-block">← Back to search</Link>
      <div className="flex gap-4 flex-col md:flex-row">
        <img src={movie.Poster !== "N/A" ? movie.Poster : "https://via.placeholder.com/300"} alt={movie.Title} className="w-64" />
        <div>
          <h1 className="text-2xl font-bold mb-2">{movie.Title}</h1>
          <p><strong>Year:</strong> {movie.Year}</p>
          <p><strong>Genre:</strong> {movie.Genre}</p>
          <p><strong>Runtime:</strong> {movie.Runtime}</p>
          <p><strong>Actors:</strong> {movie.Actors}</p>
          <p><strong>Plot:</strong> {movie.Plot}</p>
          <p><strong>IMDB Rating:</strong> {movie.imdbRating}</p>
        </div>
      </div>
    </div>
  );
}

// File: src/index.css
@tailwind base;
@tailwind components;
@tailwind utilities;
