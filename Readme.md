# The Tech Academy Live Project

## Introduction
The final two weeks of my time at The Tech Academy were spent in a real internship through Prosper I.T. Consulting, working in a team developing a full scale MVC Web Application in Microsoft Visual studio, utilizing Entity Framework code first.
During this project, I worked in a team of over 20 people, often in small groups. I got to experience using the AGILE system as well as Microsoft Azure DevOps to work effectively in a team. Each day, we held a stand-up meeting, in which each
member of the team would explain what user story they were working on, what they had accomplished the previous day, and what they were currently working on, as well as any issues they were facing. There was also a sprint planning meeting at the
beginning, as well as a retrospective at the end. I purposefully attempted to pick some diverse user stories to work on, so that I could get some experience with both the front and back end of the application. This two week sprint helped me to
develop the skills necessary to succeed as a career developer.

Though the application is not publically live, you can see below snippets of the code that I worked on.

### Issue: Duplicate Job Type Displays
The application did not have any sort of check to ensure that duplicate job entries were not posted. This was a fairly simple oversight, but important to fix. In order to fix it, I rewrote the POST request to check first to see if the entry was
valid, then if so, compare it to other job titles and job numbers. If it matched any of them, instead of POSTing, it would throw an error message displaying why it could not post. If this check failed, it would POST as normal.

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


### Issue: The dropdown list containing "work types" for employees (Foreman, Experienced MBA, etc) were listting as their internal values, creating a messy, confusing user experience
When a site admin or manager created or edited any entries involving any employees, their role in the company would be listed. This was controlled with an editable dropdown list, allowing you to give them roles such as Foreman, Experienced MBA,
New Hire, etc. Unfortunately, this dropdown menu displayed the internal values for these roles instead of clean descriptions, so you ended up having a dropdown menu with entries such as ExpMBA, and even placeholders. 

First, descriptions had to be added to the dropdown entries:

		namespace ConstructionNew.Enums
		{
			public enum WorkType
			{
				//built an unselected
				[Description("Unselected")] Unselected = 0,
				[Description("Lead Man")] LeadMan,
				[Description("Foreman")] Foreman,
				[Description("Experienced MBA")] ExpMBA,
				[Description("New MBA")] NewMBA
			}

		}

Then, a method needed to be added as an extension to the enum descriptions in order to actually use them. I also wrote a shorter method, but it seemed to be slower, so I discarded it:

		namespace ConstructionNew.Extensions
		{
			public static class EnumExtensions
			{
				public static string GetDescription(this Enum genericEnum)
				{
					Type genericEnumType = genericEnum.GetType();
					MemberInfo[] memberInfo = genericEnumType.GetMember(genericEnum.ToString());
					if ((memberInfo != null && memberInfo.Length > 0))
					{
						var _Attribs = memberInfo[0].GetCustomAttributes(typeof(System.ComponentModel.DescriptionAttribute), false);
						if ((_Attribs != null && _Attribs.Count() > 0))
						{
							return ((System.ComponentModel.DescriptionAttribute)_Attribs.ElementAt(0)).Description;
						}
					}
					return genericEnum.ToString();
				}

				public static T ParseEnum<T>(this string stringVal)
				{
					return (T)Enum.Parse(typeof(T), stringVal);
				}

			}


		}


These needed to be applied to other dropdown lists as well, just in case:

		public class ApplicationUser : IdentityUser
		{
			public async Task<ClaimsIdentity> GenerateUserIdentityAsync(UserManager<ApplicationUser> manager)
			{
				// Note the authenticationType must match the one defined in CookieAuthenticationOptions.AuthenticationType
				var userIdentity = await manager.CreateIdentityAsync(this, DefaultAuthenticationTypes.ApplicationCookie);
				// Add custom user claims here
				return userIdentity;
			}

			[Display(Name = "First Name")]
			public string FName { get; set; }
			[Display(Name = "Last Name")]
			public string LName { get; set; }
			[Display(Name = "Work Category")]
			public WorkType WorkType { get; set; }
			[Display(Name = "User Role")]
			public string UserRole { get; internal set; }
			[Display(Name = "Suspended")]
			public bool Suspended { get; set; }

			public virtual ICollection<Schedule> Schedules { get; set; }
			public virtual ICollection<ChatMessage> ChatMessages { get; set; }
		}


Here, I rewrote the partial view on the user list to get the descriptions using my method:

		<table class="table">
			<tr>
				<th>
					@Html.DisplayNameFor(model => model.UserName)
				</th>
				<th>
					@Html.DisplayNameFor(model => model.FName)
				</th>
				<th>
					@Html.DisplayNameFor(model => model.LName)
				</th>

				<th>
					@Html.DisplayNameFor(model => model.WorkType)
				</th>
				<th>
					@Html.DisplayNameFor(model => model.UserRole)
				</th>
				<th>
					@Html.DisplayNameFor(model => model.Suspended)
				</th>
			</tr>

			@foreach (var item in Model)
			{
				<tr>
					<td>
						@Html.DisplayFor(modelItem => item.UserName)
					</td>
					<td>
						@Html.DisplayFor(modelItem => item.FName)

					</td>
					<td>
						@Html.DisplayFor(model => item.LName)
					</td>
					<td>
						@using (Html.BeginForm("EditWork", "Account", FormMethod.Post, new { enctype = "multipart/form-data" }))
						{
							@Html.Hidden("Id", item.Id)
							@Html.DropDownList("workType", new SelectList(ConstructionNew.Extensions.WorkTypeMethods.GetWorkTypeDescription()), ConstructionNew.Extensions.EnumExtensions.GetDescription(item.WorkType))
							<input type="submit" value="Submit" onclick="return confirm('Click ok to change category')" />
						}
					</td>
					<td>
						@using (Html.BeginForm("EditRole", "Account", FormMethod.Post, new { enctype = "multipart/form-data" }))
						{
							@Html.Hidden("Id", item.Id)
							@Html.DropDownList("UserRole", new List<SelectListItem>
							{
								new SelectListItem { Text = "Admin", Value = "Admin"},
								new SelectListItem { Text = "Manager", Value = "Manager"},
								new SelectListItem { Text = "Employee", Value = "Employee"}
							}, item.UserRole)
							<input type="submit" value="Submit" onclick="return confirm('Click Ok to change role')" />}
					</td>
					<td>
						<!--Creates form for Suspend user and ties it to the "Suspend" method in the "AccountController" -->
						@using (Html.BeginForm("Suspend", "Account", FormMethod.Post, new { enctype = "multipart/form-data" }))
						{
							@Html.Hidden("Id", item.Id)
							<!--Creates checkbox for Suspend and binds it to the model attribute "Suspended"-->
							@Html.EditorFor(model => item.Suspended)
							{
								<input type="submit" value="Submit" onclick="return confirm('Click Ok to confirm suspension')" />}
						}
					</td>
					<td>@Html.ActionLink("Delete User", "Delete", new { id = item.Id }, new { onclick = "return confirm('Press Ok to delete');" })</td>
				</tr>
			}

		</table>


And here, I did the same with the Master Schedule partial view:

		<div class="container">
    
			@foreach (Job job in Model)
			{
				<div class="panel panel-info">
					<div class="panel-heading">
						<table style="width:100%">
							<tr>
								<td style="width:40px">
									<span class="label label-success">@job.JobNumber</span>
									</td>
								<td>
									<span style="font-weight:bold">@job.JobTitle</span>
                            
									<br />@job.StreetAddress @job.City, @job.State.GetDescription()

								</td>
								<td>
									<span>Start Time: @job.ShiftTimes</span>
								</td>
								<td>
									@*If there is a note, Display it*@
									@if (!String.IsNullOrEmpty(job.Note))
									{
										<span>Note: @job.Note</span>
									}
								</td>
								<td style="text-align:right">
									<span>
										<i class="glyphicon glyphicon-info-sign"></i>
										@*Action Link to Job Details*@
										@Html.ActionLink("Details", "Details", "Jobs", new { id = job.JobId }, null)
									</span>
								</td>
							</tr>
						</table>
					</div>
					<div class="panel-body">
						<table style="width:100%">
							<tr>
								<th>Name</th>
								<th>Work Category</th>
								<th></th>
								<th><i class="glyphicon glyphicon-calendar"> </i> Start Date</th>
								<th><i class="glyphicon glyphicon-calendar"> </i> End Date</th>
							</tr>
							@foreach (Schedule schedule in job.Schedules)
							{

								<tr style="border-bottom: 1px solid #ddd">
								<td style="width:20%">

									@*@schedule.Person.UserName Displaying username for now, perhaps delete this line later.*@
									@schedule.Person.FName @schedule.Person.LName @*Need to add First and Last names to seed data.*@


								</td>
								<td style="width:20%">
									@schedule.Person.WorkType.GetDescription() @*GetDescription method grabs the decorator statement*@
								</td>
								<td style="width:20%">
									@*Only Display Phone # for Foreman*@
									@if (schedule.Person.WorkType == WorkType.Foreman)
									{
										<i class="glyphicon glyphicon-phone-alt"> </i> @schedule.Person.PhoneNumber
									}
								</td>
								<td style="width:20%">
									@*making change for dateTime display*@
									@*@schedule.StartDate.ToString("dd/MM/yy")*@
									@schedule.StartDate.ToString("MM-dd-yyyy")@*//Modified the order month-day-year from previous for a uniform view for all views*@
								</td>
								<td style="width:20%">
									@*Enddate is Nullable, check for value first*@
									@if (schedule.EndDate.HasValue)
									{
										//Modifying Date display for uniformity
										@*@Convert.ToDateTime(schedule.EndDate).ToString("dd/MM/yy")*@
										@Convert.ToDateTime(schedule.EndDate).ToString("MM-dd-yyyy")//Modified the order month-day-year from previous for a uniform view for all views
									}
								</td>

								@*Action Link to schedule provides a quick path to EDITING for MANAGER*@
								@if (User.IsInRole("Admin") || User.IsInRole("Manager"))
								{
									<td style="text-align:right">
										<a href="@Url.Action("Edit", "Schedules", new { id = schedule.ScheduleId }, null)">
											<span class="glyphicon glyphicon-pencil" aria-hidden="true"></span>
										</a>
									</td>                   
								}
								@*Action Link to schedule provides a quick path to DETAILS for EMPLOYEE*@
								@if (User.IsInRole("Employee"))
								{
									<td style="text-align:right">
										<a href="@Url.Action("Details", "Schedules", new { id = schedule.ScheduleId }, null)">
											<span class="glyphicon glyphicon-info-sign" aria-hidden="true"></span>
										</a>
									</td>
								}
							</tr>
							}
						</table>
					</div>

				</div>
				<br />
			}
		</div>