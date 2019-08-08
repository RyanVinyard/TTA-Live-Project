# The Tech Academy Live Project

## Introduction
The final two weeks of my time at The Tech Academy were spent in a real internship through Prosper I.T. Consulting, working in a team developing a full scale MVC Web Application in Microsoft Visual studio, utilizing Entity Framework code first.
During this project, I worked in a team of over 20 people, often in small groups. I got to experience using the AGILE system as well as Microsoft Azure DevOps to work effectively in a team. Each day, we held a stand-up meeting, in which each
member of the team would explain what user story they were working on, what they had accomplished the previous day, and what they were currently working on, as well as any issues they were facing. There was also a sprint planning meeting at the
beginning, as well as a retrospective at the end. I purposefully attempted to pick some diverse user stories to work on, so that I could get some experience with both the front and back end of the application. This two week sprint helped me to
develop the skills necessary to succeed as a career developer.

Though the application is not publically live, you can see below snippets of the code that I worked on.

### Issue: Duplicate Job Type Displays
The application did not have any sort of check to ensure that duplicate job entries were not posted.

        // GET: Jobs/Create
        [Authorize(Roles = "Admin")]
        public ActionResult Create()
        {
            return View();
        }

        // POST: Jobs/Create
        // To protect from overposting attacks, please enable the specific properties you want to bind to, for 
        // more details see https://go.microsoft.com/fwlink/?LinkId=317598.
        [HttpPost]
        [ValidateAntiForgeryToken]
        [Authorize(Roles = "Admin")]
        public ActionResult Create([Bind(Include = "JobId,JobTitle,JobNumber,ShiftTimes,StreetAddress,City,State,Zipcode,Note")] Job job)
        {
            if (ModelState.IsValid)
            {
                if (db.Jobs.Any(j => j.JobTitle.Equals(job.JobTitle)))
                {
                    ModelState.AddModelError("JobTitle", "This job title is currently being used, please change the title," +
                                             " or delete/modify the job titled " + "\""+ job.JobTitle + "\".");
                    return View(job);
                }
                if (db.Jobs.Any(j => j.JobNumber.Equals(job.JobNumber)))
                {
                    ModelState.AddModelError("JobNumber", "This job number is currently being used, please change the number," +
                                             " or delete/modify the job numbered " + "\"" + job.JobNumber + "\".");
                    return View(job);
                }
                else
                {
                    job.JobId = Guid.NewGuid();
                    db.Jobs.Add(job);
                    db.SaveChanges();
                    return RedirectToAction("Index");
                }
            }
            return View(job);
        }