<script>
  import { onMount } from 'svelte';
  import axios from 'axios'
  import Alert from '../components/Alert'
  import { bepRanges, cepRanges, calculateBr, calculateCr, getFactionIcon, getFactionName } from '../statFunctions';

  onMount(() => {
  get_outfit();
  get_outfit_members();
  });

  export let params;

  let alert;
  let outfit = {};
  let outfitMembers = [];
  let url = params.id || outfit.id

  const outfitUrl = "/api/outfit/"+url
  const outfitMembersUrl = "/api/outfit/"+url+"/members"

  async function get_outfit() {
    try {
      const resp = await axios.get(outfitUrl);
      outfit = resp.data
      // Reset alert message if needed
      alert.message("");
    } catch (e) {
      console.log(e);
      alert.message("Failed to fetch stats from server");
    }
  }

  async function get_outfit_members() {
    try {
      const resp = await axios.get(outfitMembersUrl);
      const stats = resp.data;
      outfitMembers = stats.members
      // Reset alert message if needed
      alert.message("");
    } catch (e) {
      console.log(e);
      alert.message("Failed to fetch stats from server");
    }
  }

  function stripColorCode(title) {
    if (!title) return "";
    if (title.startsWith("\\#")) {
      return title.slice(8);
    }
    return title;
  }
</script>

<svelte:head>
<title>Outfit Stats</title>
</svelte:head>

<table width="70%">
  <tbody>
    <tr>
    <td width="%50" valign="top">
     <table>
      <tbody>
       <tr>
        <td>
    <img height="60" src={getFactionIcon(outfit.faction)} alt={outfit.faction}/><br>
    <span style="color:lightgrey;">Outfit Name:</span> {outfit.outfit_name}<br>
    <span style="color:lightgrey;">Created:</span>
    {new Date(outfit.created).toLocaleDateString('en-US', {
      year: 'numeric',
      month: 'long',
      day: 'numeric'
    })}<br>
    <span style="color:lightgrey;">Leader:</span> <a href="/avatar/{outfit.leader_id}">{outfit.leader_name}</a><br>
    <span style="color:lightgrey;">Points:</span> {outfit.points}<br>
    <span style="color:lightgrey;">Members:</span> {outfit.members}<br>
    </td>
  </tbody>
</table>
   </td>
   </tr>
  </tbody>
</table>
<br>
<br>
 <table class="table table-sm table-dark table-responsive-md table-striped table-hover">
  <thead class="thead-light">
      <th></th>
      <th>Name</th>
      <th>Title</th>
      <th>Points</th>
      <th>BR</th>
      <th>CR</th>
      <th>Joined</th>
  </thead>
  <tbody>
    {#each outfitMembers as member, $index}
      <tr>
        <td>{$index + 1}</td>
        <td><a href="/avatar/{member.avatar_id}">{member.avatar_name}</a></td>
        <td>{stripColorCode(member.rank_title)}</td>
        <td>{member.points}</td>
        <td>{calculateBr(member.bep)}</td>
        <td>{calculateCr(member.cep)}</td>
        <td>
        {new Date(member.joined).toLocaleDateString('en-US', {
        year: 'numeric',
        month: 'long',
        day: 'numeric'
        })}
        </td>
      </tr>
    {/each}
  </tbody>
 </table>
