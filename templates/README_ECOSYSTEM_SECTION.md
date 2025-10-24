# README.md Ecosystem Section Template

Add this section to each component repository's README.md file to link to the project ecosystem.

---

## Copy and paste the section below into your component repository's README.md (near the top):

---

## Part of AWS Services Dashboard Ecosystem

**Production Site**: https://aws-services.synepho.com

This is the **[COMPONENT NAME]** for the AWS Services Dashboard project.

**Full Project Documentation**: [aws-services-dashboard-project](https://github.com/jxman/aws-services-dashboard-project)

### Related Repositories

- **Frontend** - [aws-services-site](https://github.com/jxman/aws-services-site): React dashboard
- **Data Fetcher** - [aws-infrastructure-fetcher](https://github.com/jxman/aws-infrastructure-fetcher): Lambda data collector
- **Reporter** - [nodejs-aws-reporter](https://github.com/jxman/nodejs-aws-reporter): Excel report generator
- **Infrastructure** - [synepho-s3cf-site](https://github.com/jxman/synepho-s3cf-site): Terraform hosting
- **Documentation** - [aws-services-dashboard-project](https://github.com/jxman/aws-services-dashboard-project): Central docs

### Architecture Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  CloudWatch +   │───▶│  Data Fetcher   │───▶│   S3 Bucket     │
│  AWS Services   │    │   (Lambda)      │    │  (JSON Data)    │
└─────────────────┘    └─────────────────┘    └────────┬────────┘
                                                         │
                                                         ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Reporter      │◀───│  CloudFront +   │◀───│  React Site     │
│   (Lambda)      │    │   S3 Hosting    │    │  (Frontend)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

---

## Customize for each repository:

### For aws-services-site (Frontend):
```markdown
This is the **frontend dashboard** for the AWS Services Dashboard project.
```

### For aws-infrastructure-fetcher (Data Fetcher):
```markdown
This is the **data collection Lambda** for the AWS Services Dashboard project.
```

### For nodejs-aws-reporter (Reporter):
```markdown
This is the **Excel report generator Lambda** for the AWS Services Dashboard project.
```

### For synepho-s3cf-site (Infrastructure):
```markdown
This is the **infrastructure as code** for the AWS Services Dashboard project.
```

---

**End of template**
