# JavascriptCodeChanllenge
Javascript challenge with promise, https

`const https = require('https');

// Read Promise chapter from javascript.info to understand the concept of promise
// uncomment console.log to see the ouput at various steps
async function getTeams(year, k) {
	// API endpoint template: https://jsonmock.hackerrank.com/api/football_matches?competition=UEFA%20Champions%20League&year=<YEAR>&page=<PAGE_NUMBER>

	let totalPages = 1;
	let urls = [];

	//step 1: get the total pages in order to make that many request for finding the teams data
	let promiseURLs = new Promise(function (resolve, reject) {
		https.get(`https://jsonmock.hackerrank.com/api/football_matches?competition=UEFA%20Champions%20League&year=${year}`, (res) => {
			res.on('data', (d) => {
				let data = JSON.parse(d.toString());
				totalPages = data.total_pages;
				// console.log('Total pages', totalPages);
				for (let page = 1; page <= totalPages; page++) {
					urls.push(`https://jsonmock.hackerrank.com/api/football_matches?competition=UEFA%20Champions%20League&year=${year}&page=${page}`);
				};
				resolve(urls);
			});
		});
	});

	//step 2: on receiving the list of URLs we, send https request for each url which will eventually return data we require of matches played by teams
	promiseURLs.then((urls) => {
		// return array of promises with list of team
		return Promise.all(urls.map(url => {

			//create and return the promise of data for each urls
			return new Promise((resolve, reject) => {

				//get request for url
				https.get(url, (res) => {
					// console.log("url", url)

					let teams = [];		//empty team array for each request
					res.on('data', (d) => {
						// we need to use .toString() on d because its the data in stream/buffer format
						var result = JSON.parse(d.toString());
						result.data.forEach(e => {
							// if we have either of team empty we should not add that to match played team
							if (e.team1 != "" && e.team2 != "") {
								teams.push(e.team1);
								teams.push(e.team2);
							}
						});
					});
					// once the end event of request, fulfil the promise by resolve
					res.on('end', () => {
						// console.log('teams', teams);
						resolve(teams);
					});
				});
			});
		}));
	}).then(finalTeamsData => {
		// finalTeamsData is array of array so use destructing
		// [].concat(...Arr) will combine array of array elements into new empty array
		let teamMatchesWithCount = {};
		let finalTeams = [];
		finalTeams = finalTeams.concat(...finalTeamsData);

		//Count number of matches played by team. this is same as counting count of duplicates in array
		finalTeams.forEach((team) => {
			teamMatchesWithCount[team] = (teamMatchesWithCount[team] || 0) + 1
		});

		// console.log('team with number of matches played', teamMatchesWithCount);

		//remove duplicate teams data using Set()
		uniqueTeams = new Set(finalTeams);

		let teamWithMostMatchesPlayed = [];

		uniqueTeams.forEach(team => {
			// console.log('team', team, 'matches played', teamMatchesWithCount[team]);
			//select the teams with matches played more than k
			if (teamMatchesWithCount[team] > k)
				teamWithMostMatchesPlayed.push(team);

		});
		console.log(teamWithMostMatchesPlayed, 'teams has played greater than', k, 'matches');
	}).catch((err) => console.log(err));
}

// first argument is year and second argument is minimum number of matches to be played by team,
getTeams(2013, 3);`
