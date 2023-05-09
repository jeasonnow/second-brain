```dataviewjs
function isInThisWeek(days){
	return page => {
	  const oneDayTime = 24 * 60 * 60 * 1000;
	  const day = new Date().getDay() > 0 ? new Date().getDay() : 7;
	  const startOfDayTime = new Date(new Date().toLocaleDateString()).getTime();
	  const endOfDayTime = new Date(new Date().toLocaleDateString()).getTime() + oneDayTime - 1;
	  const MondayTime = startOfDayTime - (day - 1) * oneDayTime;
	  const SundayTime = endOfDayTime + (7 - day) * oneDayTime;
	  
	  let creationTime = Date.parse(page.file.ctime);
	  return creationTime >= MondayTime && creationTime <= SundayTime;
	}
}

dv.pages('"Calendar/Daily notes"').where(isInThisWeek()).forEach(p=> {
	dv.taskList(p.file.tasks);
})
```