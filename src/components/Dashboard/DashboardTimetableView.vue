<template>
	<div>
		<div class="content-title">Plan zajęć</div>
			<div class="timetable" style="box-shadow: 0 0 25px #ddd inset; border-radius: 5px; padding: 0.3em;"></div>
		</div>
	</div>
</template>

<script>
	import auth from '../../auth'
	export default {
		data() {
			return {
				items: []
			}
		},
		created() {
			var timetable = new Timetable()
			timetable.setScope(9, 23)

			this.$http.get('https://uek.maciekmm.net/timetable/', { headers: auth.getAuthHeader() }).then((data) => {
				var day = ['Niedziela','Poniedziałek','Wtorek','Środa','Czwartek','Piątek','Sobota']

				function dateZeros(param) {
					return param < 10 ? '0' + param : '' + param
				}

				function dateScope() {
					var current = new Date(2017, 5, 0, 12, 12, 12) //manual change for debugging
					var start = new Date(current)
					var end = new Date(current)

					start.setDate(current.getDate() - current.getDay()+1)
					start = new Date(current.getFullYear(), start.getMonth(), start.getDate())
					end.setDate(current.getDate()+7 - current.getDay()+1)
					end = new Date(current.getFullYear(), end.getMonth(), end.getDate())

					return { start: start, end: end }
				}

				function dateName(time) {
					return day[time.getDay()] + ' ' + dateZeros(time.getDate()) + '.' + dateZeros(time.getMonth()+1) + '.' + time.getFullYear()
				}

				var scope = dateScope()
				// console.log('Current date scope: ' + scope.start + ' to ' + scope.end)
				var locations = []
				var timeStart = scope.start
				
				// Create locations / labels
				for(var i = 0; i < 7; i++) {
					var time = timeStart
					if(i>0)
						time.setDate(time.getDate() + 1)
					
					var location = dateName(time)
					locations.push(location)
				}
				timetable.addLocations(locations)

				var classes = JSON.parse(data.body).classes				
				console.log(classes)				
				var calendar = []

				// Add events to the calendar
				for(var i = 0; i < classes.length; i++) {
					var item = classes[i]
					var date = new Date(item.start)
					scope = dateScope()

					if(date > scope.start && date < scope.end) {
						var c = 'block-default'
					
						switch(item.type) {
							case 'wykład':
								c = 'block-1'
								break;
							case 'ćwiczenia':
								c = 'block-2'
								break;
							case 'lektorat':
								c = 'block-3'
								break;
						}	

						var options = {
							data: item,
							class: c
						}

						console.log(item)
						timetable.addEvent(item.class, dateName(date), new Date(item.start), new Date(item.end), options)
					}
				}

				var renderer = new Timetable.Renderer(timetable);
				renderer.draw('.timetable');
			},
			(data) => {
				console.log(data.err)
			})
		}
	}
</script>

<style lang="scss" scoped>
</style>
